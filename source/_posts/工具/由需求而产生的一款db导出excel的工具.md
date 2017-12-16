---
title: 由需求而产生的一款db导出excel的工具
tags:
  - excel    
  - 工具
categories: 工具
date: 2017-12-02 10:03:05
updated: 2017-12-02 10:03:11
---
程序员最大的毛病可能就是懒,因为懒所以做出了许许多多提高自己工作效率的工具.
起因于我是商业开发,既然是商业项目避免不了各种数据统计,虽然公司有专门的数据平台,但是应对一些临时性需求还是免不了开发人员去导出一些数据.每一次有需求来我都是写一个复杂的sql,然后放到`DataGrip`中执行,利用其功能导出cvs,然而越来越多的需求该功能无法满足,比如导出组合表,也就是一个excel中有多个sheet表.那么应该这个需求我写了一个为自己的工具.

### 我理想中的工具
> 1.简单模式使用sql查询直接导出
> 2.复杂模式可以定义一些复杂的bean,然后通过组合代码中自定义实现导出逻辑
> 3.可以自己定义表头,以及对应的数据处理,比如把时间戳转换为yyy-MM-dd hh:MM:ss这样的形式
> 4.支持一个excel中含有多个sheet
> 5.不需要很复杂的配置,因为自用,所以能约定俗成的地方就约定俗成.

### 语言的选择
这个很随意了,我是选择自己最熟悉的语言,也就是Java.
同事听说我用Java写这种工具,强烈推荐我用py,但天生动态语言无感,可以说是反感,所以放弃.

### 实现
DB连接: DBUtils
Excel: POI
具体过程很简单,代码逻辑也很清晰,这里只说下主要流程,详细的可以参考源码Github地址,另外由于个人使用,所以没有太多的校验和异常考虑.

**easy-excel**   [[https://github.com/mrdear/easy-excel](https://github.com/mrdear/easy-excel)]([https://github.com/mrdear/easy-excel](https://github.com/mrdear/easy-excel))

另外分享一个IDEA从数据库表生成对应Bean的脚本,使用方法自定义自己的`extensions script`即可.
```java
import com.google.common.collect.Sets
import com.intellij.database.model.DasTable
import com.intellij.database.model.ObjectKind
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil
import org.apache.commons.lang3.StringUtils

/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */

typeMapping = [
        (~/(?i)tinyint/)                  : "Integer",
        (~/(?i)int/)                      : "Long",
        (~/(?i)float|double|decimal|real/): "Double",
        (~/(?i)date|datetime|timestamp/)  : "java.util.Date",
        (~/(?i)time/)                     : "java.sql.Time",
        (~/(?i)/)                         : "String"
]

colTypeMapping = [
        (~/(?i)tinyint/)                  : "TINYINT",
        (~/(?i)int/)                      : "BIGINT",
        (~/(?i)float|double|decimal|real/): "DECIMAL",
        (~/(?i)datetime|timestamp/)       : "TIMESTAMP",
        (~/(?i)date/)                     : "TIMESTAMP",
        (~/(?i)time/)                     : "TIMESTAMP",
        (~/(?i)text/)                     : "LONGVARCHAR",
        (~/(?i)/)                         : "VARCHAR"
]

FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable && it.getKind() == ObjectKind.TABLE }.each {
        generate(it, dir)
    }
}


def generate(table, dir) {
    def packageName = getPackageName(dir)
    def className = javaName(table.getName(), true)
    def fields = calcFields(table)
    //创建相关目录 repository目录下的
    def path = dir.toString()
        //创建pojo与xml
    new File(path + File.separator + className + ".java").withPrintWriter { out -> generatePojo(out,
                packageName, className, fields) }

}
/**
 * 生成POJO
 * @param out
 * @param packageName
 * @param className
 * @param fields
 * @return
 */
def generatePojo(out, packageName, className, fields) {

    out.println "package ${packageName};"
    out.println ""
    out.println "@Data"
    out.println "public class ${className} {"
    out.println ""
    fields.each() {
        // 输出注释
        if (StringUtils.isNoneEmpty(it.comment)) {
            out.println "  /**"
            out.println "   * ${it.comment}"
            out.println "   */"
        }
        if (it.annos != "") out.println "  ${it.annos}"
        out.println "  private ${it.type} ${it.name};"
    }
    out.println ""
    out.println "}"
}

// ------------方法 ---------------

/**
 * 拿到所有的字段
 * @param table 数据库表
 * @return 字段Object
 */
def calcFields(table) {
    DasUtil.getColumns(table).reduce([]) { fields, col ->
        def spec = Case.LOWER.apply(col.getDataType().getSpecification())
        def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
        def colTypeStr = colTypeMapping.find { p, t -> p.matcher(spec).find() }.value
        fields += [[
                           colName: col.getName(),
                           name   : javaName(col.getName(), false),
                           type   : typeStr,
                           colType: colTypeStr,
                           comment: col.getComment(),
                           annos  : ""]]
    }
}
/**
 * 获取字段对应的Java类名称
 */
def javaName(str, capitalize) {
    def s = str.split(/(?<=[^\p{IsLetter}])/).collect { Case.LOWER.apply(it).capitalize() }
            .join("").replaceAll("_", "")
    capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

/**
 * 获取包名称
 * @param dir 实体类所在目录
 * @return
 */
def getPackageName(dir) {
    def target = dir.toString().replaceAll("/", ".").replaceAll("^.*src(\\.main\\.java\\.)?", "")

    return target.charAt(0) == '.' ? target.substring(1) : target
}
```

### 总结
本文的主要目的是表达迷茫的时候不知道自己该做什么,那么就从自己身边的需求开始,分析自己所遇到的痛点,然后用你喜欢的方式去解决这个痛点,那么这个过程就是你的进步.