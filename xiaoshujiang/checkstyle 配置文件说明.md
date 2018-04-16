---
title: checkstyle 配置文件说明
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 属性说明

- basedir
代码所在的位置

- AbstractClassName
format: 定义抽象类的命名规则

- PackageName
format: 定义包名的命名规则

- TypeName
format: 定义类和接口的命名规则
tokens: 定义规则适用的类型，例如：CLASS_DEF表示类，INTERFACE_DEF 表示接口

- ParameterName
format: 定义参数名的命名规则

- ParameterNumber
max: 定义最多有多少个参数
tokens: 定义检查的类型

- StaticVariableName
format: 定义静态变量的命名规则

- MethodName
format: 定义方法名的命名规则

- LeftCurly
option: 定义左大括号'{'显示位置，eol在同一行显示，nl在下一行显示
maxLineLength: 大括号'{'所在行行最多容纳的字符数
tokens: 该属性适用的类型，例：CLASS_DEF,INTERFACE_DEF,METHOD_DEF,CTOR_DEF

- LineLength
max: 定义一行所能容许的字符数
ignorePattern: 定义可以忽略的格式

- MethodLength
检查方法的行数
max: 最多容许的行数
countEmpty: 是否计算空行
tokens: 定义检查的类型

- RightCurly
option: 右大括号是否单独一行显示
tokens: 定义检查的类型

- PackageHtml
检查对每一个包是否生成了package.html文件
fileExtensions: 指定要检查的文件的类型，如果只包含java文件，就不必指定

- JavadocType
检查类或者接口的javadoc注释
authorFormat: 检查author标签的格式
versionFormat: 检查version标签的格式
scope: 可以检查的类的范围，例如：public只能检查public修饰的类，private可以检查所有的类
excludeScope: 不能检查的类的范围，例如：public，public的类将不被检查，但访问权限小于public的类仍然会检查，其他的权限以此类推
tokens: 该属性适用的类型，例如：CLASS_DEF,INTERFACE_DEF

- JavadocMethod
检查方法的javadoc的注释
scope: 可以检查的方法的范围，例如：public只能检查public修饰的方法，private可以检查所有的方法
allowMissingParamTags: 是否忽略对参数注释的检查
allowMissingThrowsTags: 是否忽略对throws注释的检查
allowMissingReturnTag: 是否忽略对return注释的检查

- JavadocVariable
检查类变量的注释
scope: 检查变量的范围，例如：public只能检查public修饰的变量，private可以检查所有的变量

- JavadocStyle
- scope: 
- excludeScope: 
- checkFirstSentence: 
- checkEmptyJavadoc: 
- checkHtml: 
- tokens:

- LocalVariableName
format: 定义局部变量的命名规则

- LocalFinalVariableName
format: 定义局部常量的命名规则
- ConstantName
format: 定义全局常量的命名规则

- MemberName
format: 定义非静态成员变量的命名规则
applyToPublic: 是否适用于public的成员变量
applyToProtected: 是否适用于protected的成员变量
applyToPackage: 是否适用于package的成员变量
applyToPrivate: 是否适用于private的成员变量

- AvoidStarImport
必须导入类的完整路径，即不能使用*导入所需的类
excludes: 定义可以使用*导入的包

- ImportOrder
定义导入包的顺序
groups: 定义导入包的顺序，默认以字母顺序导入
ordered: 定义包是否必须按规定的顺序显示
separated: 定义包与包之间是否应添加空白行
caseSensitive: 是否区分包名的大小写

- IllegalImport
检查是否从非法的包中导入了类
illegalPkgs: 定义非法的包名称

- UnusedImports
检查是否导入的包没有使用

- RedundantImport
检查是否导入了不必显示导入的类

- EmptyForInitializerPad
检查for语句初始化变量的格式
option: 定义初始化语句中是否使用空格，例如：space表示使用空格，则for(int i = 0; i < 100; i++)就是符合格式要求的，而for(int i=0; i<100;i++)就不符合要求

- EmptyForIteratorPad
检查for iterator语句是否使用空格
option: 定义初始化语句是否使用空格，例如：space表示使用空格，则for(Iterator iterator = List.iterator(); iterator.hasNext(); iterator.next())就是形式合理的，否则就是形式不合理的

- ExecutableStatementCount
检查要执行的语句的数目
max: 定义所能容许的语句的最多数目
tokens: 定义可以检查的类型，例如：CTOR_DEF,METHOD_DEF,STATIC_INIT,INSTANCE_INIT

- FileLength
max: 定义一个文件所能容许的行数

- AnonInnerLength
检查匿名内部类
max: 定义匿名内部类最多容许的行数

- MethodParamPad
检查方法参数的格式
allowLineBreaks: 参数是否允许在不同行（注：没有作用）
option: 在参数和括号、参数和标识符之间是否包含空格

- OperatorWrap
检查运算符是否在应在同一行
option: 定义运算符的位置，eol在同一行，nl在下一行
tokens: 定义检查的类型

- ParenPad
检查左小括号'('后边和右小括号')'前边是否有空格
option: space表示有空格，nospace表示没有空格
tokens: 定义检查的类型

- TypecastParenPad
暂不清楚

- TabCharacter
检查源码中是否包含\t

- WhitespaceAfter
检查类型后是否包含空格
tokens: 检查的类型

- WhitespaceAround
暂不清楚

- ModifierOrder
检查修饰符的顺序，默认是 public,protected,private,abstract,static,final,transient,volatile,synchronized,native,strictfp（注：定义不起作用）

- RedundantModifier
检查是否有多余的修饰符，例如：接口中的方法不必使用public、abstract修饰
tokens: 检查的类型

- EmptyBlock
检查是否有空代码块
option: 定义代码块中应该包含的内容，例如：stmt表示语句
tokens: 检查的类型

- NeedBraces
检查是否应该使用括号的地方没有加括号
tokens: 定义检查的类型

- AvoidNestedBlocks
检查是否有嵌套的代码块
allowInSwitchCase: 定义是否允许switch case中使用嵌套的代码块

- ArrayTrailingComma
检查初始化数祖时，最后一个元素后面是否加了逗号，如果左右大括号都在同一行，则可以不加逗号

- AvoidInlineConditionals
检查是否在同一行初始化， 例如：private int Age = nGe==1 ? 100 : 0; 就应该避免

- CovariantEquals
暂不清楚

- ModifiedControlVariable
检查循环控制变量是否被修改

- SimplifyBooleanExpression
检查是否有boolean使用冗余的地方，例如：b == true、b || true，应该简化为 b、b

- SimplifyBooleanReturn
检查是否在返回boolean值时是否有使用冗余的地方，例如：
    if(valid())
        return true;
    else
        return false;
应该改为：
    return valid();

- StringLiteralEquality
检查在判断字符串是否相等时是否使用了正确的形式

- EqualsHashCode
检查在重写了equals方法后是否重写了hashCode方法

- FinalLocalVariable
检查变量值没有改动的情况下，该变量是否定义成了final

- MissingSwitchDefault
检查switch语句是否忘记了default标签

- RedundantThrows
检查是否抛出了多余的异常

- DefaultComesLast
检查switch中default是否在所有case的后面

- MissingCtor
检查类中是否显式定义了构造器

- FallThrough
检查switch中case后是否加入了跳出语句，例如：return、break、throw、continue

- MultipleStringLiterals
检查一个字符串变量在不改变变量值的情况下或者字符串出现的次数
allowedDuplicates: 定义在类中一个字符串变量在不改变变量值的情况下或者字符串所能使用的最多次数

- MultipleVariableDeclarations
检查一次声明多个变量时，变量是否在同一行或者在同一个语句中

- RequireThis
检查是否使用了this
checkFields: 是否检查变量引用
checkMethods: 是否检查方法调用

- UnnecessaryParentheses
检查是否使用了多余的小括号

- VisibilityModifier
检查变量是否对外部可见
packageAllowed: 变量包内成员可以访问
protectedAllowed: 变量是受保护的
publicMemberPattern: 可以公开访问的变量所匹配的命名形式

- FinalClass
只有私有构造器的类必须声明为final

- InterfaceIsType
检查接口是否只定义了变量而没有定义方法，因为接口应该用来描述一个类型，所以只定义变量而不定义方法是不恰当的
allowMarkerInterfaces: 是否检查空接口

- HideUtilityClassConstructor
只定义了静态方法的类不应该定义一个公有的构造器

- DesignForExtension
检查类是否被设计为可扩展的，如果是，则方法应该abstract、final或者是空的

- ThrowsCount
检查抛出异常的数量
max: 定义抛出异常的最大数目

- StrictDuplicateCode
检查类中是否有代码复制的问题
min: 允许代码重复的最小行数
charset: 文件所用的字符集

 - BooleanExpressionComplexity
max: 布尔运算符在一条语句中允许出现的最大数目

- GenericIllegalRegexp
检查代码中是否有不合适的引用形式或者字符
format: 定义检查所匹配的类型
ignoreCase: 是否区分大小写
ignoreComments: 是否忽略注释
message: 出现问题应该显示给用户的信息

- NewlineAtEndOfFile
检查文件是否以一个新行结束
lineSeparator: 行分隔符的类型，windows是crlf

- UncommentedMain
检查是否有没有被注掉或者删除的main方法
excludedClasses: 定义可以带main方法的类所匹配的名字形式

- UpperEll
检查初始化长整型变量时，数字後是加了大写字母'L'而不是小写字母'l'

- ArrayTypeStyle
检查再定义数组时，采用java风格还是c风格，例如：int[] num是java风格，int num[]是c风格
javaStyle: 定义是否采用java风格定义数组

- FinalParameters
检查参数是否是常量
tokens: 定义检查的类型

- Indentation
检查代码的缩进是否符合要求
basicOffset: 定义代码体相对于所属的代码体的缩进量
braceAdjustment: 定义括号的缩进量
caseIndent: 定义case的缩进量

- RequiredRegexp
检查文件中是否存在相应的文字
format: 定义所匹配的形式

- usage.OneMethodPrivateField
检查是否只有一个方法访问了私有变量
ignoreFormat: 定义可以忽略的变量所匹配的命名形式

- usage.UnusedLocalVariable
检查是否有命名後没有使用的变量
ignoreFormat: 定义可以忽略的变量所匹配的命名形式

- usage.UnusedParameter
检查是否有没有使用的参数
ignoreFormat: 定义可以忽略的变量所匹配的命名形式
ignoreCatch: 是否忽略catch中的参数
ignoreNonLocal: 是否忽略非本地的变量

- usage.UnusedPrivateField
检查是否存在未被使用的私有成员变量
ignoreFormat: 定义可以忽略的变量所匹配的命名形式

- usage.UnusedPrivateMethod
检查是否存在未被使用的私有方法
ignoreFormat: 定义可以忽略的变量所匹配的命名形式