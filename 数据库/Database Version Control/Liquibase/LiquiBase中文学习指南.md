# LiquiBase中文学习指南

> 转载：[LiquiBase中文学习指南](https://blog.csdn.net/u012934325/article/details/100652805)

领先的开源数据库更改和部署解决方案。`Liquibase` 提供独立于数据库的方式，提供快速、安全、可重复的数据库部署

## 1. 概述

此快速入门为 `Liquibase` 提供了简要指导，并涉及三个关键主题：

*   [state and Migration approaches](https://www.liquibase.org/quickstart.html#approach)
*   [How Liquibase works](https://www.liquibase.org/quickstart.html#how)
*   [Tutorials for quickly getting started with Liquibase](https://www.liquibase.org/quickstart.html#tutorials)
    *   [Tutorial: Getting Started Using SQL Scripts](https://www.liquibase.org/quickstart.html#simpleSQL)
    *   [Tutorial: Getting Started Using Liquibase Functions](https://www.liquibase.org/quickstart.html#lbmodel)

快速入门并不涵盖 `Liquibase`中提供的每个功能，而是侧重于确保您了解核心概念并能够解决基本用例。

### 1.1 数据库更改管理：状态和迁移方法

有两种方法来管理数据库更改。第一种方法是基于状态的（或声明性的）的 ， 其中定义了数据库的所需状态。可以通过对应工具将目标环境与定义的所需状态进行比对（或对比），用于生成允许目标环境与声明状态匹配的迁移脚本。另一种方法是基于迁移（或命令的），其中描述了用于更改数据库状态的特定迁移。通过对应工具能够显式跟踪和排序各个迁移，并将尚未部署到目标环境的迁移正确迁移到目标数据库。

虽然`Liquibase`能够进行比对（或对比），但它从根本上说是基于迁移的解决方案。`Liquibase`的比对能力仅用于协助加入新项目，并检查数据库迁移是否得到正确应用。作为基于迁移的解决方案，`Liquibase` 可以轻松：

*   跟踪所有建议的数据库更改，包括需要部署的特定顺序、建议/创作更改的人员，并记录更改的目的（作为注释）
    
*   清楚地回答数据库更改是否已部署到数据库。实际上，`Liquibase` 能够管理每个数据库的"版本"。
    
*   确切地将更改部署到数据库，包括将数据库提升为特定的"版本"
    
*   阻止用户修改已部署到数据库的更改，要么有意重新处理部署的更改，要么前滚。
    

### 1.2 Liquibase是如何工作的

`Liquibase`的核心是依靠一种简单的机制来跟踪、版本和部署更改：

*   `Liquibase` 使用更改日志（是更改的分类）按特定顺序显式列出数据库更改。更改日志中的每个更改都是一个`change set`。更改日志可以任意嵌套，以帮助组织和管理数据库迁移。
    *   注意： 最佳做法是确保每个`change set`都尽可能原子性更改，以避免失败的结果使数据库中剩下的未处理的语句处于unknown 状态;不过，可以将大型 `SQL` 脚本视为单个更改集。
*   Liquibase 使用跟踪表（具体称为`DATABASECHANGELOG`），该表位于每个数据库上，并跟踪已部署更改日志中的`change set`。
    *   注意：如果 `Liquibase`所在的数据库没有跟踪表，`Liquibase` 将创建一个跟踪表。
    *   注意：为了协助处理您未从空白数据库开始的项目，`Liquibase`具有生成一条更改日志以表示数据库模式当前状态的功能。

使用分类和跟踪表，`Liquibase` 能够：

*   跟踪和以版本控制数据库更改 – 用户确切知道已部署到数据库的更改以及尚未部署的更改。
*   部署更改 — 具体来说，通过将分类(`ledger`)中的内容与跟踪表中的内容进行比较，`Liquibase` 只能将以前尚未部署到数据库的更改部署到数据库中。
    *   注意：`Liquibase` 具有上下文、标签和先决条件等高级功能，可精确控制`changeSet`的部署时间以及位置。

### 1.3 教程：使用 Liquibase 跟踪、版本(`version`)和部署数据库更改

使用 `Liquibase`时，可以使用 [Liquibase 函数](https://www.liquibase.org/quickstart.html#simpleSQL)或 [SQL](https://www.liquibase.org/quickstart.html#lbmodel) 定义更改。重要的是，这些模式不是互斥的，可以结合使用，从而在如何定义和部署数据库更改方面提供了相当大的灵活性。对于使用 `Liquibase` 函数定义的更改，`Liquibase`生成适合目标数据库的`SQL`。这在下列场景是很有用的：

*   支持多个不同的数据库后端。如果您是软件供应商，希望避免编写相同的数据库迁移以支持不同的数据库平台，则这是一个常见的用例。
*   使不熟悉 `SQL` 或不熟悉 `SQL` 的开发人员能够定义数据库更改。数据库迁移可以在 `XML`、`JSON` 或 `YAML` 中定义，而不是 `SQL`。
*   标准化数据库更改。`Liquibase`将生成语法和风格上一致的`SQL`，例如，确保所有`CREATE TABLE`迁移具有相同的样式和模式。

或者，`Liquibase` 直接与用户提供的数据库迁移脚本一起使用。这在下列场景是很有用的:

*   进行不是 `Liquibase`函数的更改。自定义或特定于数据库的更改（例如，`Oracle`嵌套表）通常不是 `Liquibase`函数。
*   使精通 `SQL` 的开发人员非常倾向于直接使用 `SQL`。`Liquibase`只支持`XML` 数据库迁移，这是一个常见的误解。现实是，`Liquibase` 绝对可以支持普通的 `SQL` 脚本！

注意：`Liquibase Pro`向 `Liquibase`函数添加了用于定义过程数据库代码的更改类型。但是，与 `Liquibase` 函数的其他更改不同，这些过程数据库代码更改（如`Create Function`）需要数据库平台特定的 `SQL`（例如，在 `Oracle` 上，更改将需要 `PL/SQL`）。这些新更改类型有助于从直接检查更改日志中更好地查看特定于数据库的更改。

教程设置

在尝试任何分步教程之前，请使用设置说明准备您的环境。

#### 1.3.1 第1步：下载和解压Liquibase

1.  下载Liquibase，可以从[该页面](https://download.liquibase.org/)下载最新的二进制文件
2.  下载好`*.zip`或者`*.tar.gz`文件后，解压里面的内容到一个文件夹。

#### 1.3.2 第2步： 安装Java

1.  `Java` 是必需的依赖项。如果尚未安装 `Java`，则安装`Java`。
    
    *   注意：您可以下载并使用 [OpenJDK](https://jdk.java.net/12/)。请务必[正确配置路径和环境变量](https://stackoverflow.com/questions/52511778/how-to-install-openjdk-11-on-windows)。
2.  验证您是否具有可以运行的 `java` 版本。
    
    *   在命令行窗口执行`java -version`命令
    *   确保它可以成功运行并且显示了你安装的`Java`的版本

#### 1.3.3 第3步：下载H2 JDBC数据库驱动

1.  这些教程使用 H2 数据库。您需要下载 H2 JDBC 驱动程序，可在[此处找到](http://www.h2database.com/html/cheatSheet.html)。
2.  将 `h2_.jar`文件复制到您解压`Liquibase` `*zip` 或 `*.tar.gz`的产生的目录中。

#### 1.3.4 第4步： 设置liquibase.properties文件

1.  教程使用`CLI`。虽然可以传递所有必需的参数（如`JDBC`驱动程序和数据库`URL`），但配置 `liquibase.properties`文件会更容易节省时间和精力。
2.  创建一个 `liquibase.properties`。将以下内容添加到文件中，并将其保存在您解压`Liquibase` `*zip` 或 `*.tar.gz`的产生的目录中。

```xml
driver: org.h2.Driver
classpath: ./h2-1.4.199.jar
url: jdbc:h2:file:./h2tutorial
username: admin
password: password
changeLogFile: myChangeLog.xml

```

注意：请务必使用复制到解压的`Liquibase` 目录中的`h2_.jar`文件的实际版本！

就这些了！现在，您已设置为开始其中一个教程！

### 1.4 Liquibase 教程：开始使用 SQL 脚本

#### 1.4.1 第1步创建一个sql文件夹

在解压的`Liquibase` 的文件夹中 ，创建一个 `sql` 文件夹。在这个文件夹中你将放置 `Liquibase`将跟踪、版本和部署的`SQL`脚本。

#### 1.4.2 第2步建立一个Change Log

这是一次性步骤，用于配置更改日志以指向将包含 `SQL` 脚本的 `sql` 文件夹。在解压的`*.zip` 或`*.tar.gz`的 `Liquibase` 的目录中创建并保存文件名为 `myChangeLog.xml` 的文件 。`myChangeLog.xml` 的内容应如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <includeAll path="/sql"/>
</databaseChangeLog>

```

#### 1.4.3 第3步在SQL文件中新增一个SQL脚本

使用教程设置中的 `liquibase.properties` 文件以及新创建的`myChangeLog.xml`，我们现在已准备好开始向 `sql`文件夹添加`SQL`脚本。`Liquibase` 将在文件夹中按字母数字顺序排列脚本。使用以下内容创建 `001_create_person_table.sql` 并将其保存在 `sql`文件夹中：

```sql
create table PERSON (
    ID int not null,
    FNAME varchar(100) not null
);

```

#### 1.4.4 第4步 部署你的第一个修改

现在，我们已准备好部署我们的第一个脚本！打开终端，如果在 `UNIX`系统上则运行 `./liquibase update`或 如果在 Windows 上则运行`liquibase.bat update`。

#### 1.4.5 第5步 检查你的数据库

您将看到您的数据库现在包含一个名为`PERSON`的表。要将作为本教程一部分的 `H2` 数据库写入内容，请打开一个终端，导航到您提取的 `Liquibase``*.zip` 或 `*.tar.gz`的文件夹，并运行 `java -jar h2-1.4.199.jar`注意：输入您下载的 `h2*.jar` 的特定版本！输入`JDBC URL`、用户名和密码，从 `liquibase.properties` 文件输入您根据教程设置创建的属性文件。您会注意到还创建了另外两个表：`databasechangeloglock`和`databasechangeloglock`。`databasechangelog`表包含针对数据库运行的所有更改的列表。`databasechangeloglock`表用于确保两台计算机不会同时尝试修改数据库。

#### 1.4.6 后续步骤

*   此快速入门指南旨在快速向您介绍 `Liquibase`。有关其所有功能的完整描述，请参阅`Liquibase` [手册](http://www.liquibase.org/documentation/index.html)，阅读[最佳实践](https://www.liquibase.org/bestpractices.html)并访问[论坛](http://www.liquibase.org/community/index.html)。
    
*   如果您有一个现有项目，您正在寻找添加`Liquibase`，请访问[现有项目](https://www.liquibase.org/documentation/existing_project.html)页面。
    
*   如果您对商业支持、培训或咨询感兴趣，请考虑 [Liquibase Pro](https://download.liquibase.org/)。
    
* * *

### 1.5 教程：使用 Liquibase 函数入门

本教程使用`Liquibase`函数。更改将在 `XML` 中定义，而不是使用 `SQL`。`Liquibase` 将根据定义的`changeSet(s)`生成 `SQL`，并将该 `SQL` 部署到目标数据库。所有迁移都在更改日志中显式跟踪和排序。

#### 1.5.1 第1步创建一个Changelog文件

database changelog文件是列出所有数据库更改的位置。创建一个名为 `myChangeLog.xml`的文件，其中包含以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

</databaseChangeLog>

```

#### 1.5.2 第2步 添加一个Change Set

每个`changeSet`都由`id`属性和`author`属性进行唯一标识。这两个标记以及更改日志文件的名称和包唯一地标识了更改。如果只需要指定一个`id`，则很容易意外重用它们，尤其是在处理多个开发人员和代码分支时。包括`author`属性可最大程度地减少重复的可能性。

将每个`changeSet`视为要应用于数据库的原子更改。通常最好在`changeSet`中只包含一个更改，但如果插入多个行时，这些行一起有作为单个事务添加的意义，则允许进行更多更改。`Liquibase` 将尝试将每个`changeSet`运行为单个事务，但对于某些命令，许多数据库将静默地提交和回滚事务（创建表、删除表等）。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <changeSet id="1" author="bob">
        <createTable tableName="department">
            <column name="id" type="int">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="name" type="varchar(50)">
                <constraints nullable="false"/>
            </column>
            <column name="active" type="boolean" defaultValueBoolean="true"/>
        </createTable>
    </changeSet>

</databaseChangeLog>

```

#### 1.5.3 第3步 运行change Set

如果是在`UNIX`操作系统，可以使用`./liquibase update`命令来执行`myChangeLog.xml`；如果是在`Windows`操作系统上，可以使用`liquibase.bat update`来运行`myChangeLog.xml`

#### 1.5.4 检查你的数据库

您将看到您的数据库现在包含一个名为`PERSON`的表。要将作为本教程一部分的 `H2` 数据库入内容，请打开一个终端，导航到您提取的 `Liquibase``*.zip` 或 `*.tar.gz`的文件夹，并运行 `java -jar h2-1.4.199.jar`注意：输入您下载的 `h2*.jar` 的特定版本！输入`JDBC URL`、用户名和密码，从 `liquibase.properties` 文件输入您根据教程设置创建的属性文件。您会注意到还创建了另外两个表：`databasechangeloglock`和`databasechangeloglock`。`databasechangelog`表包含针对数据库运行的所有更改的列表。`databasechangeloglock`表用于确保两台计算机不会同时尝试修改数据库。

#### 1.5.5 后续步骤

*   此快速入门指南旨在快速向您介绍 `Liquibase`。有关其所有功能的完整描述，请参阅`Liquibase` [手册](http://www.liquibase.org/documentation/index.html)，阅读[最佳实践](https://www.liquibase.org/bestpractices.html)并访问[论坛](http://www.liquibase.org/community/index.html)。
    
*   如果您有一个现有项目，您正在寻找添加`Liquibase`，请访问[现有项目](https://www.liquibase.org/documentation/existing_project.html)页面。
    
*   如果您对商业支持、培训或咨询感兴趣，请考虑 [Liquibase Pro](https://download.liquibase.org/)。
    
* * *

## 2. Liquibase 最佳实践

本篇介绍了可以应用于项目的许多最佳实践。

### 2.1 组织你的changeLogs

组织`changeLogs`的最常见方法是按主要版本。在`classpath`中选择一个包来存储`changeLogs`，最好是在数据库访问类附近。在此示例中，我们将使用 `com/example/db/changlog`.

#### 2.1.1 目录结构如下所示

```java
com
  example
    db
      changelog
        db.changelog-master.xml
        db.changelog-1.0.xml
        db.changelog-1.1.xml
        db.changelog-2.0.xml
      DatabasePool.java
      AbstractDAO.java

```

**db.changelog-master.xml**

`master.xml` 包括版本以正确顺序排列的`changelog`。在上面的示例中，它可能如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                      http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <include file="com/example/db/changelog/db.changelog-1.0.xml"/> 
  <include file="com/example/db/changelog/db.changelog-1.1.xml"/> 
  <include file="com/example/db/changelog/db.changelog-2.0.xml"/> 
</databaseChangeLog> 

```

每个包含的 `XML` 文件需要与标准 `XML` 数据库`changelog`的格式相同，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<databaseChangeLog 
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.9" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.9
                      http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd"> 
  <changeSet author="authorName" id="changelog-1.0">
    <createTable tableName="TablesAndTables">
      <column name="COLUMN1" type="TEXT">
        <constraints nullable="true" primaryKey="false" unique="false"/>
      </column>
    </createTable>
  </changeSet>
</databaseChangeLog> 

```

`db.changelog-master.xml`是您传递给所有 `Liquibase` 调用的`changelog`。

#### 2.1.2 管理存储过程

尝试为存储过程维护单独的`changelog`，并使用 `runOnChange="true"`。此标志强制 `LiquiBase` 检查`changeSet`是否被修改。如果是这样，`liquibase` 将再次执行更改。

#### 2.1.3 每个`changeSet`只做一个更改

尽可能避免每个`changeSet`进行多次更改，以避免可能使数据库处于意外状态的自动提交语句失败。

#### 2.1.4 `ChangeSet`的 `ids`

选择适合您的内容。有些使用从 1 开始的序列号，在`changelog`中唯一，有些选择描述性名称（例如`new-address-table`）。

#### 2.1.5 文档的`ChangeSets`

在`changeSet`中使用`<comments>`,`they say : a stitch in time saves nine`

#### 2.1.6 始终考虑回滚

尝试以可以回滚的方式编写`changeSet`。例如，使用相关的更改子句，而不是使用自定义`<sql>`标记。每当更改不支持开箱即用回滚时，请包含`<rollback>`子句。（例如`<sql>`、`<insert>`等）

#### 2.1.7 参考数据管理

利用 `Liquibase` 管理您的参考数据。环境分离（`DEV`、`QA`、`PROD`）可以使用`context`实现。

#### 2.1.8 开发人员的过程

*   使用您最喜爱的`IDE` 或编辑器，创建一个新的本地`changeSet`包含了更改。
    
*   运行 `Liquibase` 以执行新的`changeSet`（这将测试 `SQL` 代码）;
    
*   对应用程序代码执行相应的更改（例如，Java 代码）;
    
*   测试新的应用程序代码以及数据库更改;
    
*   提交`changeSet`和应用程序代码。
    

### 2.2 考虑`Datical`

*   `Datical` 是一种基于 `Liquibase` 核心功能的商业产品。除了版本控制和管理数据库更改之外，`Datical` 还通过启用数据库代码完全统一和自动化路径的功能来弥合开发和操作之间的差距。
    
*   `Datical` 具有 `Web` 界面、命令行界面和 `REST API`。所有接口都是安全的，需要身份验证。
    
*   `Datical`根据组织标准自动验证数据库代码，以消除手动审核
    
*   `Datical` 自动从经过验证的 `DDL`代码生成`changeSet`，从而消除了在编写`changeSet`和手动更新`changeSet`时手动工作。
    
*   `Datical` 为数据库代码生成一个不可变的项目，用于一致、可重复和自动化就绪的下游部署
    
*   `Datical` 使用目标数据库的基于对象的模型预测数据库更改的影响，以确保在部署数据库更改时，`Datical` 与票务系统（如 `JIRA`）集成时没有错误或问题，以便轻松地将数据库更改回溯到源代码和初始要求。此功能还便于保存或加速功能集
    
*   `Datical` 可以快照和比较数据库架构，以帮助识别和`address drift`
    

想了解更多关于`Datical`的信息, 你可以参考该网站 [http://www.datical.com/Liquibase](http://www.datical.com/liquibase/)

### 2.3 `Liquibase`的选择

对于数据库源代码管理需求，有两种不同的选项可供选择。

#### 2.3.1  `Liquibase`社区版本

开源数据库版本控制。`Liquibase` 为团队提供了一个很好的起点，以应对管理数据库更改带来的挑战。它比推送数据库脚本的作用更大，它还生成和部署它们。

[社区版本下载链接](https://download.liquibase.org/download-community/)

#### 2.3.2 `Liquibase`企业版

虽然 `Liquibase`社区是一个很好的起点，但希望如何充分利用`Liquibase`的最佳实践，以及随着解决方案的扩展而提高可见性和控制力的团队可能会发现 `Liquibase Pro` 更适合他们的需求。`Liquibase Pro` 增加了增强和扩展基本开源功能的能力，例如添加函数以更改用于更新过程数据库代码的集。非试用 `Liquibase Pro`许可证附带支持，因此您不必单独使用。免费试用14天。无需信用卡。

[Liquibase Pro版本收费目录](https://download.liquibase.org/liquibase-pro-pricing-details/)

### 2.4 核心概念

#### 2.4.1 `Changelog` 文件

开发人员将数据库更改存储在其本地开发计算机上基于文本的文件中，并将其应用于其本地数据库。`Changelog`文件可以任意嵌套，以便更好地管理。

[更多](http://www.liquibase.org/documentation/databasechangelog.html)

#### 2.4.2 `ChangeSet`

`changeSet`由`author`和`id`属性以及`changelog`文件的位置唯一标识，是 `Liquibase` 跟踪执行的单位。运行 `Liquibase` 时，它会查询标记为已执行的`changSet`的`DATABASECHANGELOG` 表，然后执行更改日志文件中尚未执行的所有`changeSet`。

[更多](http://www.liquibase.org/documentation/changeset.html)

#### 2.4.3 `Changes`

更改每个`changeSet`通常包含一个更改，该更改描述要应用于数据库的更改/重构。`Liquibase` 支持为支持的数据库和原始 `SQL` 生成 `SQL` 的描述性更改。通常，每个`changeSet`应只有一个更改，以避免可能使数据库处于意外状态的自动提交语句失败。

[更多](http://www.liquibase.org/documentation/changes/index.html)

#### 2.4.4 `Preconditions`

先决条件可以应用于整个`changelog`或单个`changeSet`。如果先决条件失败，`liquibase`将停止执行。

#### 2.4.5 `Contexts`

可以将上下文应用于`changeSet`，以控制在不同环境中运行的`changeSet`。例如，某些`changeSet`可以标记为`production`，另一些可以标记为`test`。如果未指定上下文，则无论执行上下文如何，`changset`都将运行。

[更多](http://www.liquibase.org/documentation/contexts.html)

### 2.5 数据库`Change Log`文件

所有 `Liquibase` 更改的根目录是`databaseChangeLog`文件。

#### 2.5.1 可用属性

`logicalFilePath`:用于在创建`changeSet`的唯一标识符时覆盖文件名和路径。移动或重命名`change logs`时是必需的。

#### 2.5.2 可用的子标签

*   `preConditions`:执行更改日志所需的先决条件。[read more](http://www.liquibase.org/documentation/preconditions.html)
*   `property`:将属性设置为的值（如果不是通过其他方法设置）。[read more](http://www.liquibase.org/documentation/changelog_parameters.html)
*   `changeSet`:要执行的`changeSet`。[read more](http://www.liquibase.org/documentation/changeset.html)
*   `include`:包含要执行的`changeSet`的其他文件。[read more](http://www.liquibase.org/documentation/include.html)

当 `Liquibase` 迁移器运行时，它将分析数据库 `ChangeLog` 标记。它首先检查指定的先决条件。如果先决条件失败，`Liquibase`将退出，并显示一条错误消息，解释失败的原因。先决条件对于记录和强制执行更改日志编写器的预期和假设（如要针对的 `DBMS` 或以用户身份运行更改）非常有用。

如果满足所有的先决条件，`Liquibase`将会开始运行在`databaseChangeLog`文件中按照顺序出现`changeSet`和`include`标签。

`databaseChangeLog`标签的`XML`模式在3.1就可以使用：

[dbchangelog-3.1.xsd](http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd)

某些旧版 XSD 列在[XML Format page](http://www.liquibase.org/documentation/xml_format.html)

每个`changeSet`包含一个`id`标签和一个`author`标签。这些标签以及类路径位置和 `XML` 文件的名称为该更改集创建唯一标识符。

**示例：空的`change log`**

**XML Format**

```xml
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd
    http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">
</databaseChangeLog>

```

**YAML Format**

```yaml
databaseChangeLog:

```

**JSON Format**

```json
{
    "databaseChangeLog": [
    ]
}

```

**SQL Format**

```sql


```

可以附加`preConditions`到`databaseChangeLog`或`changeSet`，以控制基于数据库状态的更新的执行。

下面是使用`preConditions`的几个原因:

*   记录更改日志的编写者在创建`changelog`时的假设。
*   强制使运行`change log`的用户不会违反这些假设
*   在执行不可恢复的更改（如 `drop_Table`）之前执行数据检查
*   根据数据库的状态控制哪些`changeSet`运行

如果需要，`preConditions`可以是`<changeSet>`中的唯一标记。

`databaseChangeLog`级别的`preConditions`适用于所有`changeSet`，而不仅仅是当前`changeLog`或其子`changelog`中列出的`changeset`。

**下面是简单使用`preConditions`的示例**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.8"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.8
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.8.xsd">
    <preConditions>
        <dbms type="oracle" />
        <runningAs username="SYSTEM" />
    </preConditions>

    <changeSet id="1" author="bob">
        <preConditions onFail="WARN">
            <sqlCheck expectedResult="0">select count(*) from oldtable</sqlCheck>
        </preConditions>
        <comment>Comments should go after preCondition. If they are before then liquibase usually gives error.</comment>
        <dropTable tableName="oldtable"/>
    </changeSet>
</databaseChangeLog>

```

仅当针对 `Oracle`执行的数据库和执行脚本的数据库用户为`SYSTEM`时，才会运行上述`databasechangelog`。仅当"oldtable"中没有值时，它才会运行 `drop_Table`命令。

### 2.6 处理失败和错误

自 1.8 版本起，`Liquibase` 区分了`preConditions`的"失败"（检查失败）和"错误"（在执行检查时引发的异常），并且可以通过`<preConditions>`标签上的`onFail`和`onError`属性控制对两者的反应。

#### 2.6.1 可用的属性

*   `onFail`: 当`proConditions`遇到失败的时候如何处理
*   `onError`:当`proConditions`遇到错误的时候如何处理
*   `onUpdateSQL`:自版本1.9.5后当`proConditions`遇到更新`SQL`模型的时候如何处理
*   `onFailMessage`:自2.0起，在`proConditions`失败时要输出的自定义消息。
*   `onErrorMessage`:在`proConditions`错误时要输出的自定义消息。

#### 2.6.2 `onFail`或者`onError`可能的取值

*   `HALT`:立即停止执行整个`changelog`。默认的值。
*   `CONTINUE`:跳过`changeset`。将在下次更新时再次尝试执行`changeset`。继续`changelog`。
*   `MARK_RAN`:跳过`changeset`，但将其标记为已执行。继续`changelog`。
*   `WARN`:输出警告并继续正常执行`changeset`/`changelog`。

在`changset`之外（例如，在`changelog`的开头），可能的值只有 `HALT`和 `WARN`。

#### 2.6.3 `onUpdateSQL`可能的值

*   `RUN`:在`upateSQL`的模式下运行`changeSet`
*   `FAIL`:在`updateSQL`的模式下运行`preConditions`
*   `IGNORE`:在`updateSQL`的模式下忽略`preConditions`

### 2.7 `AND/OR/NOT`逻辑

可以使用可嵌套`<and>`、`<or>`和`<not>`标签将条件逻辑应用于`preConditions`。如果未指定条件标签，则默认为 `<AND>`。

例子：

```xml
 <preConditions onFail="WARN">
     <dbms type="oracle" />
     <runningAs username="SYSTEM" />
 </preConditions>

```

将检查更新是否运行在`Oracle`上，并且用户是`SYSTEM`,但是如果`preConditions`失败，它将只会生成警告信息。

```xml
 <preConditions>
     <dbms type="oracle" />
     <dbms type="mysql" />
 </preConditions>

```

要求运行在`Oracle`和`MySQL`上，这总是虚假的，除非发生大规模和意外的合并。

```xml
<preConditions>
     <or>
         <dbms type="oracle" />
         <dbms type="mysql" />
     </or>
 </preConditions>

```

需要在`Oracle` 或`MySQL` 上运行，这比上面的示例更有意义。

```xml
 <preConditions>
     <or>
         <and>
            <dbms type="oracle" />
            <runningAs username="SYSTEM" />
         </and>
         <and>
            <dbms type="mssql" />
            <runningAs username="sa" />
         </and>
     </or>
 </preConditions>

```

如果对 `Oracle` 数据库执行，则需要以 `SYSTEM`身份运行;如果对`MS-SQL` 数据库运行，则需要以 `SA`身份运行。

### 2.8 可用的`preConditions`

`<dbms>`:

*   如果针对所执行的数据库与指定的类型匹配，则通过。
*   `type`:预期的[数据库](https://www.liquibase.org/databases.html)类型。可以使用逗号分隔值指定多个 `dbms` 值。**必填**

`<runningAs>`

*   如果执行的数据库用户与指定的用户名匹配，则通过。
*   `username`:数据库用户脚本应以原样运行。**必填**

`<columnExists>`

*   从1.8开始如果数据库中存在具体的列，则通过
*   `schemaName`:表的`schema`的名称。**必填**
*   `tableName`:列表的名称。**必填**
*   `columnName`:列名称。**必填**

`<tableExists>`

*   从1.8开始，如果数据库中存在具体的表，则通过
*   `schemaName`:表的`schema`的名称。**必填**
*   `tableName`:表的名称。**必填**

`<viewExists>`

*   从1.8开始，如果数据库中存在具体的视图，则通过
*   `schemaName`:视图的`schema`的名称。**必填**
*   `viewName`:视图的名称。**必填**

`<foreignKeyConstrainExists>`

*   从1.8开始，如果数据库存在指定的外键，则通过
*   `schemaName`:外键的`schema`名称，**必填**
*   `foreignKeyName`:外键的名称。**必填**

`<indexExists>`

*   从1.8开始，如果数据库存在指定的索引，则通过
*   `schemaName`:索引的`schema`名称，**必填**
*   `indexName`:索引名称,**必填**

`<sequenceExists>`

*   从1.8开始，如果数据库存在指定的序列，则通过
*   `schemaName`:序列的`schema`名称，**必填**
*   `sequenceName`:序列的名称，**必填**

`<primaryKeyExists>`

*   从1.8开始，如果数据库中存在指定的主键，则通过
*   `schemaName`:主键的`schema`名称
*   `primaryKeyName`:主键的名称，**表名或者主键名是必填**
*   `tableName`:包含主键的表的名称。从1.9开始**表名或者主键名是必填**

`<sqlCheck>`

*   执行 `SQL` 字符串并检查返回的值。`SQL` 必须返回具有单个值的单个行。要检查行数，请使用`SQL` 函数`count`。要检查值范围，请在 `SQL` 中执行检查，并返回一个可以容易比较的值。
    
*   ```xml
    <sqlCheck expectedResult="1">
        SELECT COUNT(1) FROM pg_tables      WHERE TABLENAME = 'myRequiredTable'
    </sqlCheck>
    
    ```
    
*   `expectedResult`:这个值与`SQL`的执行结果作比较，**必填**
    

`<changeLogPropertyDefined>`

*   检查是否存在给定的[`changelog`参数](http://www.liquibase.org/documentation/changelog_parameters#property)。如果还给定了值，则仅当该值与给定值不同时，该值才会失败。
    
*   从2.0开始
    
*   ```xml
    <changeLogPropertyDefined property="myproperty"/>
    <changeLogPropertyDefined property="myproperty" value="requiredvalue"/>
    
    ```
    
*   `property`:要检验的属性的名称，**必填**
    
*   `value`:给定属性的必需值。
    

`<customPrecondition>`:

*   可以通过创建实现 [`liquibase.precondition.CustomPrecondition`](http://www.liquibase.org/javadoc/liquibase/precondition/CustomPrecondition.html)接口的类来创建自定义`precondition`。自定义类上的参数通过基于`<param>`子标签的反射进行设置。参数作为字符串传递到自定义`preCondition`。
    
*   ```xml
    <customPrecondition className="com.example.CustomTableCheck">
        <param name="tableName" value="our_table"/>
        <param name="count" value="42"/>
    </customPrecondition>
    
    ```
    
*   `className`:`custom precondition`类的名称。**必填**
    
*   子标签
    
    *   `param`:传递给`custom precondition的参数`
        *   `param`子标签属性:
            *   `name`:要设置的参数的名称。**必填**
            *   `value`:要将参数设置为的字符串值。**必填**

### 2.9 实施说明

`preConditions`在执行特定`changelogs`开始时进行检查。如果使用`include`标签，并且仅在子`changelog`上具有`preConditions`，则在迁移器到达该文件之前不会检查这些`preConditions`。此行为可能会在将来的版本中更改，因此不要依赖此行为。

### 2.10 `Drop Table`

删除已经存在的表

### 2.11 可用的属性

*   `cascadeConstraints`: 支持所有
*   `catalogName`:支持所有；`catalog`的名称;从3.0开始有该属性
*   `schemaName`:支持所有；`schema`的名称
*   `tableName`:支持所有；要删除的表的名称;

**XML 例子**

```xml
<changeSet author="liquibase-docs" id="dropTable-example">
    <dropTable cascadeConstraints="true"
            catalogName="cat"
            schemaName="public"
            tableName="person"/>
</changeSet>

```

**YAML例子**

```yaml
changeSet:
  id: dropTable-example
  author: liquibase-docs
  changes:
  - dropTable:
      cascadeConstraints: true
      catalogName: cat
      schemaName: public
      tableName: person

```

**JSON例子**

```json
{
  "changeSet": {
    "id": "dropTable-example",
    "author": "liquibase-docs",
    "changes": [
      {
        "dropTable": {
          "cascadeConstraints": true,
          "catalogName": "cat",
          "schemaName": "public",
          "tableName": "person"
        }
      }]
    
  }
}

```

**从上面的示例生成 `SQL`（`MySQL`）**

```sql
DROP TABLE cat.person;

```

### 2.12 支持的数据库

图片liquibase0002

### 2.13 Change Log 参数

**从Liquibase 1.7开始**

`Liquibase` 允许在`changelog`中动态替换参数。使用 `${}` 语法描述要替换的参数。

#### 2.13.1 配置参数的值

参数的值会被按照下面的顺序进行查找

*   作为参数传递给您的 Liquibase 运行程序（请参阅 [`Ant`](http://www.liquibase.org/documentation/ant/index.html)、[`command`](http://www.liquibase.org/documentation/command_line.html)等文档，了解如何传递它们）**ant和command没有了解**
*   作为 `JVM`系统属性
*   在数据库`ChangeLog`文件本身的参数块（`<property>`标签）中
*   作为环境变量

例子：

```xml
<createTable tableName="${table.name}">
     <column name="id" type="int"/>
     <column name="${column1.name}" type="varchar(${column1.length})"/>
     <column name="${column2.name}" type="int"/>
</createTable>

```

```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <property name="clob.type" value="clob" dbms="oracle"/>
    <property name="clob.type" value="longtext" dbms="mysql"/>

    <changeSet id="1" author="joe">
         <createTable tableName="${table.name}">
             <column name="id" type="int"/>
             <column name="${column1.name}" type="${clob.type}"/>
             <column name="${column2.name}" type="int"/>
         </createTable>
    </changeSet>
</databaseChangeLog>

```

#### 2.13.2 `<property>`

给`changelog`定义一个参数。给定上下文`and/or`数据库的列表，该参数将仅用于这些上下文`and/or`数据库。

**可用属性**

*   `name`:表`schema`的名称；**必填**
*   `value`:列表的名称；**必填**
*   `context`:以逗号分隔列表表示的上下文。
*   `dbms`:作为逗号分隔列表给出的数据库类型。
*   `global`:定义属性是全局的还是仅限于数据库`ChangeLog`的。以`true`或`false`表示。

例子：

```xml
<property name="simpleproperty" value="somevalue"/>
<property name="clob.type" value="clob" dbms="oracle,h2"/>
<property name="clob.type" value="longtext" dbms="mysql"/>
<property name="myproperty" value="yes" context="common,test"/>
<property name="localproperty" value="foo" global="false"/>

```

* * *

#### 2.13.3 `<changeSet>`标签

`changeSet`标签是你用来分组数据库更改的。[refactoring](https://www.liquibase.org/documentation/changes/index.html)

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.7"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.7
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.7.xsd">
    <changeSet id="1" author="bob">
        <comment>A sample change log</comment>
        <createTable/>
    </changeSet>
    <changeSet id="2" author="bob" runAlways="true">
        <alterTable/>
    </changeSet>
    <changeSet id="3" author="alice" failOnError="false" dbms="oracle">
        <alterTable/>
    </changeSet>
</databaseChangeLog>

```

每个`changeSet`标签都由`id`标签、`author`标签和`changelog`的`classpath`名称的组合唯一标签。`id` 标签仅用作标识符，它不指示更改运行的顺序，甚至不一定是整数。如果您不知道或不希望保存实际作者，只需使用占位符值，如`UNKNOW`。

当 `Liquibase`执行数据库`ChangeLog`时，它按顺序读取`changeSet`，并针对每个`changeSet`检查`databasechangelog`表，以查看是否运行了 `id/author/filepath`的组合。如果已运行，则将跳过`changeSet`，除非存在真正的`runAlways`标签。运行`changeSet`中的所有更改后，`Liquibase` 将在`databasechangelog`中插入带有 `id/author/filepath`的新行以及`changeSet`的`MD5Sum`（见下文）。

`Liquibase` 尝试执行每个`changeSet`并在每次结束时提交事务，或者如果出现错误，则回滚。某些数据库将自动提交语句，这些语句会干扰此事务设置，并可能导致意外的数据库状态。因此，通常最好每个`changeSet`只进行一次更改，除非有一组非自动提交更改要应用为事务（如插入数据）。

**可用属性**

*   `id`:字母数字标识符，**必须**
*   `author`:创建`changeSet`的人，**必须**
*   `dbms`:要用于`changSet`的数据库的类型。运行迁移步骤时，它会根据此属性检查数据库类型。[有效的数据库类型名称列在受支持的数据库页上](https://www.liquibase.org/databases.html)
*   `runAlways`:执行每次运行时设置的更改，即使更改之前已运行
*   `runOnChange`:在第一次看到更改时以及每次更改集更改时执行更改
*   `context`:如果在运行时传递了特定上下文，则执行更改。任何字符串都可用于上下文名称，并且它们处于不区分大小写状态。
*   `runInTransaction`:`changeSet`是否应作为单个事务运行（如果可能）？默认值为 `true`。从1.9开始，警告：小心使用此属性。如果设置为 `false`，并且通过运行包含多个语句的 `changeSet`部分发生错误，则 `Liquibase` 数据库更改日志表将保持无效状态。
*   `failOnErroe`:如果在执行`changeSet`时发生错误，是否认为此迁移失败？

#### 2.13.4 可用的子标签

*   `comment`:`changeSet`的说明。`XML` 注释将提供相同的好处，`Liquibase` 的未来版本可能能够利用`<comment>`标记注释来生成文档
*   `preConditions`:将执行`changeSet`之前必须通过的前提条件。可用于在做不可恢复的内容（如自 1.7 起删除表）之前执行数据健全性检查
*   `<AnyRefactoringTag(s)>`:作为此`changeSet`的一部分运行的数据库更改（称为重构）
*   `validCheckSum`:列出被认为对此更改有效的校验，而不考虑数据库中存储的内容。自 1.7 起,主要用于需要修改`changeSet`，并且不希望在已运行过此修改的数据库上引发错误（不是建议的步骤）。
*   `rollback`:描述如何[回滚](https://www.liquibase.org/documentation/rollback.html)`changeSet`的 SQL 语句或重构标签

**Rollback标签**

回滚标记描述如何对 `SQL` 语句、`change tag`或以前的`changeSet`的引用进行回滚更改。

回滚标签的例子：

```xml
<changeSet id="1" author="bob">
    <createTable tableName="testTable">
    <rollback>
        drop table testTable
    </rollback>
</changeSet>

```

```xml
<changeSet id="1" author="bob">
    <createTable tableName="testTable">
    <rollback>
        <dropTable tableName="testTable"/>
    </rollback>
</changeSet>

```

```xml
<changeSet id="2" author="bob">
    <dropTable tableName="testTable"/>
    <rollback changeSetId="1" changeSetAuthor="bob"/>
</changeSet>

```

### 2.14 `ChangeSet`校验总结

当 `Liquibase` 达到 `changeSet` 时，它会计算一个校验和并将其存储在`databasechangelog`中。存储校验总和的值是使 `Liquibase` 可以知道是否有人更改了自运行以来更改了`changeSet`。如果`changeSet`自运行以来发生更改，`Liquibase` 将退出迁移并报错，因为它无法知道更改了哪些内容，并且数据库的状态可能与`changelog`所期望的状态不同。如果`changeSet`有正当理由已更改，并且要忽略此错误，请更新`databasechangelog`表，以便具有相应 `id/author/filepath`的行具有校验总值的空值。下次 `Liquibase` 运行时，它将将校验总和值更新为新的正确值。

校验和还与`runOnChange` changeSet属性结合使用。有时您可能不希望添加新的`changSet`，因为您只需了解当前版本，但您希望在更新此更改时应用此更改。存储过程是何时需要此方法的一个好示例。如果每次进行更改时将存储过程的整个文本复制到新的`changeSet`，您不仅会获得很长的`changelog`，而且还会失去源代码管理的融合和分散功能。而是将存储过程的文本放在具有 `runOnChange="true"`属性的`changeSet`中。仅当存储过程的文本发生更改时，才会重建存储过程。

### 2.15 回滚`ChangeSet`

`Liquibase` 允许您自动或通过自定义回滚 `SQL` 撤消对数据库所做的更改。回滚支持在`command_line`、`Ant`和 [`Maven`](https://www.liquibase.org/documentation/maven/index.html)中可用。**Maven没有看**

#### 2.15.1 如何控制回滚 SQL

许多重构（如`create table`、`rename column`和`add column`）可以自动创建回滚语句。如果您的`changeLog`仅包含适合此类别的语句，则将自动生成回滚命令。

其他重构（如`drop table`和`insert data`）没有可自动生成的相应回滚命令。在这些情况下，如果要覆盖默认生成的回滚命令，可以通过`changeSet`标签中的标签指定回滚命令。如果不希望执行任何操作来撤消回滚模式下的更改，请使用空标签。

**可用的回滚标签属性**

*   `nested tags`:标准的`Liquibase`更改标签来生成回滚语句
*   `tag text`:使用`sql`去运行回滚更改
*   `changeSetId`:要重新运行的`changeSet`的 `ID` 以回滚此更改。示例：要回滚 `Drop Table` 更改，请参阅创建表的`changeSet`。
*   `changeSetAuthor`:要重新运行的`changeSet`的作者，以便回滚此更改

**样例**

很多`change`标签是不需要`rollback`标签的，因为通过`update`语句，他们可以自动生成。

```xml
<changeSet id="changeRollback2-create" author="nvoxland">
     <createTable tableName="changeRollback2">
         <column name="id" type="int"/>
     </createTable>
</changeSet>

```

标准的`change`标签可以被使用`<rollback>`标签

```xml
   <changeSet id="changeRollback" author="nvoxland">
        <createTable tableName="changeRollback1">
            <column name="id" type="int"/>
        </createTable>
        <rollback>
            <dropTable tableName="changeRollback1"/>
        </rollback>
    </changeSet>

```

多个语句可以被包含在一个`<rollback>`标签中。一个`changeSet`中可以有多个`rollback`标签

```xml
<changeSet id="multiRollbackTest" author="rs">
    <createTable tableName="multiRollback1">
        <column name="id" type="int"/>
    </createTable>
    <createTable tableName="multiRollback2">
        <column name="id" type="int"/>
    </createTable>
    <createTable tableName="multiRollback3">
        <column name="id" type="int"/>
    </createTable>
    <rollback>
        drop table multiRollback1;
        drop table multiRollback2;
    </rollback>
    <rollback>drop table multiRollback3</rollback>
</changeSet>

```

`rollback`标签可以引用更改设置最初创建的语句

```xml
<changeSet id="changeRollback2-drop" author="nvoxland">
     <dropTable tableName="changeRollback2"/>
     <rollback changeSetId="changeRollback2-create" changeSetAuthor="nvoxland"/>
</changeSet>

```

如果无法/需要回滚，回滚标记可以为空

```xml
<changeSet id="noRollback" author="nvoxland">
    <sql>insert into multiRollback3 (id) values (1)</sql>
    <rollback/>
</changeSet>

```

#### 2.15.2 `Roll Back to`模式

您可以通过以下三种方式指定要回滚的更改：

*   `Tag`:指定要回滚到的标签将回滚标签后针对目标数据库执行的所有`changeSet`。有关如何标记数据库，请参阅[`command line`](https://www.liquibase.org/documentation/command_line.html)文档。
*   `change Sets`的数量：您可以指定要回滚的`changeSet`数。
*   `Date`:您可以指定要回滚到的日期。

#### 2.15.3 回滚执行模式

`Liquibase` 有三种管理回滚的模式：

*   直接执行回滚：回滚命令可以直接针对目标数据库执行。在无法回滚任何更改时，您将会收到通知，并且不会回滚任何更改。
*   生成一个回滚脚本：与实际更新数据库相比，可以生成回滚数据库所需的 `SQL`。在实际运行之前，如果要预览将执行哪些回滚命令，此模式将非常适合。
*   生成一个“未来回滚”脚本：此模式旨在允许您在生成迁移脚本的同时生成回滚脚本。它允许您采用更新的应用程序，并生成 `SQL` 以将数据库更新到新版本，以及在需要时生成 `SQL` 以将新版本恢复到当前版本。当 `DBA`希望控制 `SQL`进入数据库时，以及对于需要内部`and/or` `SOX-compliant`流程的回滚文档的应用程序，此功能非常有用。在此模式下，不需要指定回滚日期、标记或计数。

* * *

### 2.16 `XML`格式化

`Liquibase` 支持 `XML`作为存储`changelog`文件的格式。

#### 2.16.1 `XSD`支持

`XSD`架构定义可用于每个 `Liquibase` 版本。由于修补程序版本中没有`changelog`格式，因此只有 `xsd` 文件对应于主要、次要版本。

*   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd
*   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.0.xsd
*   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd
*   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd
*   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.8.xsd

#### 2.16.2 `Liquibase`的扩展`XSDs`

如果您使用的是包含其他更改标签的 `Liquibase` 扩展，请查看扩展文档以了解它们是否提供了 `XSD`。如果不这样做，则可以使用 xsd 在http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd允许任何嵌套标记和属性。

#### 2.16.3 限制

无

#### 2.16.4 例子

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.0.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <preConditions>
        <runningAs username="liquibase"/>
    </preConditions>

    <changeSet id="1" author="nvoxland">
        <createTable tableName="person">
            <column name="id" type="int" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="firstname" type="varchar(50)"/>
            <column name="lastname" type="varchar(50)">
                <constraints nullable="false"/>
            </column>
            <column name="state" type="char(2)"/>
        </createTable>
    </changeSet>

    <changeSet id="2" author="nvoxland">
        <addColumn tableName="person">
            <column name="username" type="varchar(8)"/>
        </addColumn>
    </changeSet>
    <changeSet id="3" author="nvoxland">
        <addLookupTable
            existingTableName="person" existingColumnName="state"
            newTableName="state" newColumnName="id" newColumnDataType="char(2)"/>
    </changeSet>
</databaseChangeLog>

```

### 2.17 YAML 格式化

`Liquibase` 支持 [YAML](https://yaml.org/) 作为存储`changelog`文件的格式。

#### 2.17.1 要求

要使用基于 `YAML` 的更改日志，必须在类路径中包括 [snakeyaml-1.12.jar](https://bitbucket.org/asomov/snakeyaml/)。

#### 2.17.2 限制

无

#### 2.17.3 例子

```yaml
databaseChangeLog:
  - preConditions:
    - runningAs:
        username: liquibase

  - changeSet:
      id: 1
      author: nvoxland
      changes:
        - createTable:
            tableName: person
            columns:
              - column:
                  name: id
                  type: int
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: firstname
                  type: varchar(50)
              - column:
                  name: lastname
                  type: varchar(50)
                  constraints:
                    nullable: false
              - column:
                  name: state
                  type: char(2)

  - changeSet:
      id: 2
      author: nvoxland
      changes:
        - addColumn:
            tableName: person
            columns:
              - column:
                  name: username
                  type: varchar(8)

  - changeSet:
      id: 3
      author: nvoxland
      changes:
        - addLookupTable:
            existingTableName: person
            existingColumnName:state
            newTableName: state
            newColumnName: id
            newColumnDataType: char(2)

```

### 2.18 JSON 格式化

`Liquibase` 支持`JSON`作为存储`changelog`文件的格式。

#### 2.18.2 要求

要使用基于 `JSON`的更改日志，必须在类路径中包括 [snakeyaml-1.12.jar](https://bitbucket.org/asomov/snakeyaml/)。

#### 2.18.3 限制

无

#### 2.18.4 例子

```json
{
    "databaseChangeLog": [
        {
            "preConditions": [
                {
                    "runningAs": {
                        "username": "liquibase"
                    }
                }
            ]
        },

        {
            "changeSet": {
                "id": "1",
                "author": "nvoxland",
                "changes": [
                    {
                        "createTable": {
                            "tableName": "person",
                            "columns": [
                                {
                                    "column": {
                                        "name": "id",
                                        "type": "int",
                                        "autoIncrement": true,
                                        "constraints": {
                                            "primaryKey": true,
                                            "nullable": false
                                        },
                                    }
                                },
                                {
                                    "column": {
                                        "name": "firstname",
                                        "type": "varchar(50)"
                                    }
                                },
                                {
                                    "column": {
                                        "name": "lastname",
                                        "type": "varchar(50)",
                                        "constraints": {
                                            "nullable": false
                                        },
                                    }
                                },
                                {
                                    "column": {
                                        "name": "state",
                                        "type": "char(2)"
                                    }
                                }
                            ]
                        }
                    }
                ]
            }
        },

        {
            "changeSet": {
                "id": "2",
                "author": "nvoxland",
                "changes": [
                    {
                        "addColumn": {
                            "tableName": "person",
                            "columns": [
                                {
                                    "column": {
                                        "name": "username",
                                        "type": "varchar(8)"
                                    }
                                }
                           ]
                        }
                    }
                ]
            }
        },

        {
            "changeSet": {
                "id": "3",
                "author": "nvoxland",
                "changes": [
                    {
                        "addLookupTable": {
                            "existingTableName": "person",
                            "existingColumnName":"state",
                            "newTableName": "state",
                            "newColumnName": "id",
                            "newColumnDataType": "char(2)",
                        }
                    }
                ]
            }
        }
    ]
}

```

### 2.19 格式化`SQL` `changelogs`

自 `Liquibase 2.0` 起，`Liquibase` 包括支持普通 `SQL` `changelog`文件。这些`changelog`可能包含在 `XML` 的`changelog`中，并且可能包含任意 `SQL` 语句。语句将转换为自定义 `sql` 重构。

格式化的 `SQL` 文件使用注释为 `Liquibase` 提供元数据。每个 `SQL` 文件必须以以下注释开头：

```sql


```

#### 2.19.1 `ChangeSets`

格式化的 `SQL` 文件中的每个`changeSet`都以注释形式开头

```sql


```

`changeset`注释后跟一个或多个 `SQL` 语句，用分号（或`<endDelimiter>`属性的值）分隔。

**可用的`changeset`属性**

每个`changeSet`上可能提供以下属性：

*   `scriptComments`: 设置为true可以在sql执行之前移除所有注释，false则相反，默认为true。
*   `splitStatements`:设置为false时，在“s”和“go”上不会使用Liquibase 拆分语句，默认为true。
*   `endDelimiter`:设置语句结尾的分隔符。默认为";“可以设置为”"
*   `runAlways`:每次运行的时候都执行此changeSet，即使之前执行过。
*   `runOnChange`:在第一次看到更改时以及每次更改集更改时执行更改
*   `context`:如果在运行时传递了特定上下文，则执行更改。任何字符串都可用于上下文名称，并且它们处于不区分大小写状态。
*   `logicalFilePath`:用于在创建`changeSet`的唯一标识符时覆盖文件名和路径。移动或重命名`change logs`时是必需的。 --和下方重复，有点问题
*   `labels`:labels是将changeSet分到context的通用方法，但是与在运行时定义一组context，然后在changeSet中定义一个匹配表达式相反，是定义好context的一组labels后运行时匹配对应表达式。
*   `runInTransaction`:changeSet是否应作为单个事务运行（如果可能的情况）？默认为true。警告：注意这个属性。如果设置为false，并且在运行包含多个语句的changeSet的过程中发生错误，Liquibase DatabaseChangeLog表将使它们处于无效状态。
*   `failOnError`:如果在执行变更集时发生错误，是否认为此迁移失败？
*   `dbms`:要用于`changSet`的数据库的类型。运行迁移步骤时，它会根据此属性检查数据库类型。[有效的数据库类型名称列在受支持的数据库页上](https://www.liquibase.org/databases.html)
*   `logicalFilePath`:

`Preconditions`

可以为每个`changeSet`指定`preConditions`。目前，仅支持 `SQL`检查`preConditions`。

```sql



```

回滚行为

`changset`可能包括回滚`changeSet`时要应用的语句。回滚语句是窗体的注释

```sql


```

简单的`Change Log`

```sql



create table test1 (
    id int primary key,
    name varchar(255)
);



insert into test1 (id, name) values (1, ‘name 1′);
insert into test1 (id, name) values (2, ‘name 2′);


create sequence seq_test;

```

* * *

#### 2.19.2 `Custom SQL`

`sql`标签允许您指定所需的任何 `sql`。它对于 `Liquibase` 的自动重构标记不支持的复杂更改非常有用，并且可用于解决 `Liquibase` 的错误和限制。`sql` 标签中包含的 `SQL` 可以是多行的。

创建过程重构是创建存储过程的最佳方式。

`sql`标签还可以支持同一文件中的多行语句。在 `SQL` 的最后一行的末尾，语句可以使用 `;`或者go拆分 。多行 `SQL` 语句也受支持，并且仅支持 `;`或go语句结束，另起新行是不够的。包含单个语句的文件不需要使用`;`或者go。

`sql` 更改还可以包含以下任一格式的注释：

多行注释可以以`/*`开始，以`*/`结束。一个单行注释可以以`<space>`开头在语句的末尾以`<space>`结束；注意：默认的它企图在语句的末尾拆分语句按照`;`或者`go`。因此，如果您有注释或其他非语句结尾`;`或`go`，不要在行尾有它，否则您将获得无效的 `SQL`。

可用的属性

*   `comment`:
*   `dbms`:
*   `endDelimiter`:要应用于语句末尾的分隔符。默认值为`;`，可以设置为`''`。
*   `splitStatement`:设置为 `false`，以上没有 `iquibase` 拆分语句`;`和`GO`。如果未设置，则默认值为 `true`
*   `sql`:
*   `stripComments`:设置为 `true`以在执行之前删除 `SQL`中的任何注释，否则为 `false`。如果未设置，则默认值为`false`

**XML 例子**

```xml
<changeSet author="liquibase-docs" id="sql-example">
    <sql dbms="h2, oracle"
            endDelimiter="\nGO"
            splitStatements="true"
            stripComments="true">insert into person (name) values ('Bob')
        <comment>What about Bob?</comment>
    </sql>
</changeSet>

```

**YAML 例子**

```yaml
changeSet:
  id: sql-example
  author: liquibase-docs
  changes:
  - sql:
      comment: What about Bob?
      dbms: h2, oracle
      endDelimiter: \nGO
      splitStatements: true
      sql: insert into person (name) values ('Bob')
      stripComments: true

```

**JSON 例子**

```json
{
  "changeSet": {
    "id": "sql-example",
    "author": "liquibase-docs",
    "changes": [
      {
        "sql": {
          "comment": "What about Bob?",
          "dbms": "h2, oracle",
          "endDelimiter": "\\nGO",
          "splitStatements": true,
          "sql": "insert into person (name) values ('Bob')",
          "stripComments": true
        }
      }]
    
  }
}

```

从上面的示例生成`SQL`（`MySQL`）

```sql
insert into person (name) values ('Bob')
GO

```

### 2.20 数据库支持

图片Liquibase0003

### 2.21 其它形式的`changelog`

除了内置的[`XML`](https://www.liquibase.org/documentation/xml_format.html)、[`YAML`](https://www.liquibase.org/documentation/yaml_format.html)、[`JSON`](https://www.liquibase.org/documentation/json_format.html)和[`SQL`](https://www.liquibase.org/documentation/sql_format.html) `changelog`格式外，`Liquibase`扩展系统还允许您以任何你喜欢的格式创建`changelog`文件。

其他社区管理的格式包括：

*   [Groovy Liquibase](https://github.com/tlberglund/groovy-liquibase)
*   [Clojure Liquibase Wrapper](https://github.com/kumarshantanu/clj-liquibase)

* * *

#### 2.21.1 Contexts

`Liquibase` 中的`context`是可以添加到`changeSet` 的标签，以控制将在任何特定的迁移运行中执行哪些设置。任何字符串都可用于上下文名称，并且它们处于不区分大小写状态。

通过任何可用方法运行迁移器时，可以传递一组要运行的`context`。将仅运行使用已传递上下文标记的`changeset`。

在你迁移的时候如果没有指定一个具体的`context`，那么所有的`context`都会执行。

下面是一个在`change`标签中使用`context`属性的例子：

```xml
<changeSet id="2" author="bob" context="test">
        <insert tableName="news">
            <column name="id" value="1"/>
            <column name="title" value="Liquibase 0.8 Released"/>
        </insert>
        <insert tableName="news">
            <column name="id" value="2"/>
            <column name="title" value="Liquibase 0.9 Released"/>
        </insert>
    </changeSet>

```

`Context`语法

`context`语法具体是使用`AND`,`OR`,`!`和圆括号。在没有圆括号的情况下，操作的顺序是`!`,`AND`,`OR`

**例子**

*   `context="!test"`
*   `context="v1.0 or map"`
*   `context="v1.0 or map"`
*   `context="!qa and !master"`

使用`,`来分隔上下文的工作方式类似于 `OR` 操作，但具有最高的优先级。

**例子**

*   `"test,qa"`等价于`test OR qa`
*   `"test,qa and master"`等价于`test OR (qa AND master)`

**有效性**

*   在`Liquibase`的所有版本中`,`分隔符都是可用的
*   在3.2.0版本中新增了`AND`,`OR`,`!`和圆括号

**使用上下文测试数据**

如果您使用 `Liquibase` 管理测试数据，最佳方式是与所有其他`changeSet`放在一起，但是加上标有`test`的`context`。这样，当您希望插入测试数据时，可以使用`context`为`test`运行迁移器。在迁移生产数据库时，不要包含`context='test'`，并且不会包含测试数据。如果您有多个测试环境或测试数据集，只需使用不同的上下文（如`mini-test`、`integration-test`等）标记它们即可。  
使用上下文控制测试数据比使用单独的changelog树要好，因为重构和更改后需要使用现有的测试数据，就像它们也会使用生产数据一样。如果有一组测试数据是在数据库建立后创建并简单添加的，那么您需要不断地手动更新测试数据脚本，以使它们与当前数据库模式保持一致。

**使用上下文进行多 `DBMS` 更改日志**

可以使用上下文来控制哪些`changeSet`在哪些数据库上运行，但更好的选择是在`changeSet` 标记上使用内置的`dbms`标记。

**默认的上下文**

从 `Liquibase 3.5` 开始，您可以在`databaseChangeLog`节点中指定`context`属性，以在默认情况下将该上下文分配给`changelog`中的所有`changeSet`。  
特别的context需要在文件中的changeSet中特别指定，它会已AND的形式接在context后面。

**`include/includeAll` 上下文**

从 `Liquibase 3.5` 开始，您可以指定一个`context`属性使用 or 标签。如果指定，则给定的context将添加到包含文件中的所有`changeSet`。

* * *

`includeAll`标签允许你将你的`change-logs`分解为很多个来管理，它和`include`标签很类似，但是不是传递一个特定的`changelog`文件来包含，你指定一个目录，它将会把这个目录中的所有`*.xml`文件当做`changelog`文件，所有的`*.sql`文件作为单独的更改。所有被发现的文件都会按照字母顺序依次执行的。

**例子**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.9"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.9
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd">
    <includeAll path="com/example/changelogs/"/>
</databaseChangeLog>

```

**警告**

虽然 `includeAll` 标签有许多有价值的用途，但其使用可能会导致问题。要避免的最大事情是使用 `includeAll` 标签来模拟按文件顺序运行的更改列表的 `Ruby` 活动迁移策略，每个文件一个。虽然这看起来是个好主意,但是它很快就会[运行出现问题](http://www.liquibase.org/2007/06/the-problem-with-rails-active-migrations.html)

如果选择使用 includeAll 标记，请确保有一个命名策略，以确保您永远不会发生冲突，或者需要重命名文件以强制重新排序

* * *

#### 2.21.2 `include` 标签

include标签允许您将 change-logs分解为几个更易于管理的部分。如果需要更容易地包含多个文件，请使用includeall标记。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.9"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.9
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd">
    <include file="com/example/news/news.changelog.xml"/>
    <include file="com/example/directory/directory.changelog.xml"/>
</databaseChangeLog>

```

随着项目的增长，`changelog`中的`changesets`数量可能会变得难以操作。为了帮助缓解此问题，并使更改管理更容易，可以包括数据库`changelog`来创建`changelog`树。在上面的示例中，根`changelog`首先包括 `com/example/news/news.changelog.xml` 中的更改，然后包括 `com/example/directory/directory.changelog.xml` 中的更改。

包含的`changelog`，因此确实需要小心谨慎，以确保包含的`changelog`是完全独立的，或者确保首先运行任何必需的`changelog`。

在运行任何`changeSet`之前，将考虑在子`changelog`文件中的`changelog`级别定义的任何`preConditions`。

使用`<include>`标签而不是使用 `XML` 的内置包含功能的原因是，使用内置功能，解析器只能看到一个大型 `XML` 文档。我们使用 `id`、`author`和`filename`唯一标识每个`change`，因此您只需要确保每个文件中的 `id/author`组合是唯一的，而不是跨所有`changelog`。

**可用的属性**

*   `file`:要`include`进来的文件的名称，**必填**
*   `relativeToChangelogFile`:**从1.9开始**，是相对于根`changelog`文件而不是类路径的文件路径。默认值为`false`

**实施说明**

检查是否有循环`changelog`或双包含`changelog`。

如果包含两次`changelog`，则不是问题，因为第二次，`Liquibase` 会知道该`changeset`已运行，并且不会再次运行它们（即使有运行始终标记）。

如果创建一个`changelog`循环 （`root.changelog.xml`包含`news.changelog.xml`，其中包括 `root.changelog.xml`），您将获得无限循环。检查循环是我们增强功能列表中的一项功能，但目前尚未实现。

### 2.22 生成`change logs`

当开始在现有数据库上使用 `Liquibase` 时，使用生成`changelog`以创建当前数据库架构通常很有用，尤其是对于测试。`Liquibase` 允许您使用`generateChangeLog` `command_line` 命令执行此操作。

请注意，此命令当前有一些限制。它不导出以下类型的对象：

*   存储过程，函数和包
*   触发器

**例子**

```properties
liquibase --driver=oracle.jdbc.OracleDriver \
      --classpath=\path\to\classes:jdbcdriver.jar \
      --changeLogFile=com/example/db.changelog.xml \
      --url="jdbc:oracle:thin:@localhost:1521:XE" \
      --username=scott \
      --password=tiger \
      generateChangeLog

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.1"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.1
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.1.xsd">
    <changeSet author="diff-generated" id="1185214997195-1">
        <createTable name="BONUS">
            <column name="ENAME" type="VARCHAR2(10,0)"/>
            <column name="JOB" type="VARCHAR2(9,0)"/>
            <column name="SAL" type="NUMBER(22,0)"/>
            <column name="COMM" type="NUMBER(22,0)"/>
        </createTable>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-2">
        <createTable name="DEPT">
            <column name="DEPTNO" type="NUMBER(2,0)"/>
            <column name="DNAME" type="VARCHAR2(14,0)"/>
            <column name="LOC" type="VARCHAR2(13,0)"/>
        </createTable>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-3">
        <createTable name="EMP">
            <column name="EMPNO" type="NUMBER(4,0)"/>
            <column name="ENAME" type="VARCHAR2(10,0)"/>
            <column name="JOB" type="VARCHAR2(9,0)"/>
            <column name="MGR" type="NUMBER(4,0)"/>
            <column name="HIREDATE" type="DATE(7,0)"/>
            <column name="SAL" type="NUMBER(7,2)"/>
            <column name="COMM" type="NUMBER(7,2)"/>
            <column name="DEPTNO" type="NUMBER(2,0)"/>
        </createTable>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-4">
        <createTable name="SALGRADE">
            <column name="GRADE" type="NUMBER(22,0)"/>
            <column name="LOSAL" type="NUMBER(22,0)"/>
            <column name="HISAL" type="NUMBER(22,0)"/>
        </createTable>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-5">
        <addForeignKeyConstraint baseColumnNames="DEPTNO"
            baseTableName="DEPT" constraintName="FK_NAME"
            referencedColumnNames="DEPTNO" referencedTableName="EMP"/>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-6">
        <createIndex indexName="PK_DEPT" tableName="DEPT">
            <column name="DEPTNO"/>
        </createIndex>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-7">
        <createIndex indexName="PK_EMP" tableName="EMP">
            <column name="EMPNO"/>
        </createIndex>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-8">
        <addPrimaryKey columnNames="DEPTNO" tableName="DEPT"/>
    </changeSet>
    <changeSet author="diff-generated" id="1185214997195-9">
        <addPrimaryKey columnNames="EMPNO" tableName="EMP"/>
    </changeSet>
</databaseChangeLog>

```

### 2.22 数据库文档生成器

在已经存在的数据库中的将更改信息存储到`changelog`中，`Liquibase`可以生成数据库更改文档

#### 2.22.1 运行`DBDoc`

`dbDoc`命令支持当前仅通过[`command line`](https://www.liquibase.org/documentation/command_line.html)可用

#### 2.22.2 样例

```properties
liquibase.sh --driver=oracle.jdbc.OracleDriver \
        --url=jdbc:oracle:thin:@testdb:1521:test \
        --username=bob \
        --password=bob \
        --changeLogFile=path/to/changelog.xml
    dbDoc \
        /docs/dbdoc

```

#### 2.22.3 样本输出

文档输出基于`JavaDoc`样式文档。[此处](http://www.liquibase.org/dbdoc/index.html)提供了示例`changelog`中的更改报告。

### 2.23 更新数据库

`Liquibase` 允许您应用您和其他开发人员已添加到`changelog`文件中的数据库更改。

#### 2.23.1 如何跟踪更改集状态

每个`changeSet`都有一个`id`和`author`属性，该属性与`changelog`文件的目录和文件名一起唯一标识它。

`Liquibase` 按顺序读取`changelog`文件中的`changeSet`，并将标识符与存储在`DatabaseChangeLog`表中的值进行比较。如果表中不存在标识符，则运行`changeSet`，并将新行添加到包含标识符和`changeSet`的 `MD5Sum`哈希的数据库`changelog`表中。

如果该标识符已存在于databasechangelog表中，则将当前存在的change set的md5sum与数据库中的md5sum进行比较。如果它们不同，Liquibase将抛出一个错误，警告您有人意外更改了它，或者根据runOnChange changeset属性的状态重新执行它。

#### 2.23.2 控制更新

应用未运行`changeset`有两种模式：

*   `update`应用于所有的未被运行的更改
*   `updateCount`仅应用给定数量的未运行更改。

#### 2.23.3 SQL 更新模式

可以将所需的 [`SQL` 存储](https://www.liquibase.org/documentation/sql_output.html)以供审阅和后续应用程序使用，而不是将`changeSet`直接应用于数据库。

### 2.24 SQL 输出

根据您的开发和发布过程，您可能不希望`Liquibase` 直接更新您的数据库。原因可能包括希望调整生成的 `SQL`、让 `DBA`批准`SQL`或实现 [`SOX` 合规性](http://www.liquibase.org/2007/07/sox-compliance-and-database-refactoring.html)。 因此，所有更新和回滚命令都有`sql 输出`模式，该模式不针对数据库执行任何操作，而是将生成的 `SQL` 保存到标准输出或指定文件。

### 2.25 SOX 合规性和数据库重构

管理、跟踪和应用数据库更改是非常困难的，尤其是在敏捷数据库环境中，在项目的整个生命周期中发生了许多更改。即使使用 `Liquibase` 这样的工具，以一致且可追溯的方式应用数据库更改需要大量训练。

对于需要处理符合`SOX`版本的项目，该过程更加困难，因为发布文档不仅需要包括如何更新数据库，还需要包括如何在发布出现问题时回滚。

为了解决这一问题，我们增加了对`Liquibase`的自动回滚支持。对于`changelog`文件中的每个更改设置，`Liquibase`可以（通常）生成回滚 `SQL`。对于无法自动撤消的更改（`drop table`、插入数据等），或者如果要重写默认回滚方法，可以指定包含正确 `SQL` 的标签。这种生成回滚命令的方法效果很好，因为在大多数情况下，您不必执行任何操作，并且当您必须指定回滚 `SQL` 时，它存储在原始更改的一侧。

要控制生成 `SQL`以更新数据库并回滚数据库的生成，请参阅`command line migrator`文档。

### 2.26 `DATABASECHANGELOG` 表

`Liquibase` 使用 `DATABASEchangeLOG` 表来跟踪已运行的`changeSet`。

该表将每个更改设置作为一行进行跟踪，由存储`changelog`文件的路径的`id`、`author`和`filename`列的组合标识。

| 列              | 标准数据类型   | 描述                                                         |
| --------------- | -------------- | ------------------------------------------------------------ |
| `ID`            | `VARCHAR(255)` | `changeSet`中的`id`属性值                                    |
| `AUTHOR`        | `VARCHAR(255)` | `changeSet`中的`author`属性值                                |
| `FILENAME`      | `VARCHAR(255)` | `changelog`的路径。这可能是一个绝对路径或一个相对路径，具体取决于`changelog`如何传递到 `Liquibase`。为获得最佳结果，它应该是一个相对路径 |
| `DATEEXECUTED`  | `DATETIME`     | 执行`changeSet`的日期/时间。与 `ORDEREXECUTED` 一起使用以确定回滚顺序 |
| `ORDEREXECUTED` | `INT`          | 执行`changeSet`的顺序。除 `DATE EXECUTED`外，还用于确保顺序正确，即使数据库日期时间支持较差的精度也是如此。 注： 仅在单个更新运行中保证值增加。有时，它们会在零处重新启动。 |
| `EXECTYPE`      | `VARCHAR(10)`  | `changeSet`是如何执行的描述。可能的值有`EXECUTED`, `FAILED`, `SKIPPED`, `RERAN`, 和 `MARK_RAN` |
| `MD5SUM`        | `VARCHAR(35)`  | 执行`changeSet`时的校验和。用于每次运行，以确保`changelog`文件中的 `changSet` 没有意外更改 |
| `DESCRIPTION`   | `VARCHAR(255)` | `changeSet`生成的可读的描述                                  |
| `COMMENTS`      | `VARCHAR(255)` | `changeSet`的`comment`标签的值                               |
| `TAG`           | `VARCHAR(255)` | `changeSet`的跟踪对应于标签操作。                            |
| `LIQUIBASE`     | `VARCHAR(20)`  | 用于执行`changeSet`的 `Liquibase` 版本                       |

注意事项

该表没有主键。这是为了避免对密钥长度进行特定于数据库的任何限制。

### 2.27 `DATABASECHANGELOGLOCK`表

`Liquibase` 使用 `DATABASEchangeLOGLOG` 表确保一次只运行一个 `Liquibase` 实例。

因为`Liquibase` 只是从 `DATABASEchangeLOG` 表读取以确定需要运行的`changeSet`，因此，如果同时对同一数据库执行多个 `Liquibase`实例，则会发生冲突。如果多个开发人员使用相同的数据库实例，或者集群中有多个服务器在启动时自动运行 `Liquibase`，则可能会发生这种情况。

| 列            | 标准数据类型   | 描述                                                         |
| ------------- | -------------- | ------------------------------------------------------------ |
| `ID`          | `INT`          | 锁的 ID。目前只有一个锁，但将来是有用的                      |
| `LOCKED`      | `INT`          | 如果 `Liquibase`正在针对此数据库运行，则设置为"1"。否则设置为"0" |
| `LOCKGRANTED` | `DATETIME`     | 获取锁的日期和时间                                           |
| `LOCKEDBY`    | `VARCHAR(255)` | 被谁锁住的描述                                               |

注意事项

如果 `Liquibase` 未干净地退出，则锁住的行可能会保留为锁定状态。您可以通过运行`UPDATE DATABASECHANGELOGLOCK SET LOCKED=0`清除当前锁

### 2.28 集成`JEE CDI`

`Liquibase`可以通过实现多种`CDI Procducers` 方法在 [`JEE CDI`](http://weld.cdi-spec.org/)环境中运行。`CDI Liquibase`集成是一个简单的 `CDI`扩展，在 `CDI`容器启动时执行`Liquibase` 更新。

#### 2.28.1 如何配置`Liquibase`

```java

public class LiquibaseProducer {

    @Resource
    private DataSource myDataSource;

    @Produces @LiquibaseType
    public CDILiquibaseConfig createConfig() {
        CDILiquibaseConfig config = new CDILiquibaseConfig();
        config.setChangeLog("liquibase/parser/core/xml/simpleChangeLog.xml");
        return config;
    }

    @Produces @LiquibaseType
    public DataSource createDataSource() throws SQLException {
        return myDataSource;
    }

    @Produces @LiquibaseType
    public ResourceAccessor create() {
        return new ClassLoaderResourceAccessor(getClass().getClassLoader());
    }

}

```

#### 2.28.2 `CDILiquibaseConfig` 可用属性

*   changeLog
*   contexts
*   parameters
*   defaultSchema
*   dropFirst **从2.0.2版本开始**

如果您不希望 `Liquibase` 运行，您可以配置以下系统属性`liquibase`应该`run=false`，

### 2.29 `Servlet Listener`

`Liquibase`可以在`Servlet Listener`中运行。无论何时你的站点被部署，`Liquibase`都允许你的数据库自动更新。因为`Liquibase`采用的是分布式锁，即使您有一个应用程序服务器群集，此方法也能正常工作。

为了配置`Servlet Listener`，需要将`liquibase.jar`添加到`WEB-INF/lib`目录下，你的`web.xml`的内容如下：

```xml
<context-param>
    <param-name>liquibase.changelog</param-name>
    <param-value>com/example/db.changelog.xml</param-value>
</context-param>

<context-param>
    <param-name>liquibase.datasource</param-name>
    <param-value>java:comp/env/jdbc/default</param-value>
</context-param>

<context-param>
    <param-name>liquibase.host.includes</param-name>
    <param-value>production1.example.com, production2.example.com</param-value>
</context-param>

<context-param>
    <param-name>liquibase.onerror.fail</param-name>
    <param-value>true</param-value>
</context-param>

<context-param>
    <param-name>liquibase.contexts</param-name>
    <param-value>production</param-value>
</context-param>

<listener>
    <listener-class>liquibase.integration.servlet.LiquibaseServletListener</listener-class>
</listener>

```

如果使用的`liquibase`版本是1.9，`web.xml`应该添加如下内容：

```xml
<context-param>
    <param-name>LIQUIBASE_CHANGELOG</param-name>
    <param-value>com/example/db.changelog.xml</param-value>
</context-param>

<context-param>
    <param-name>LIQUIBASE_DATA_SOURCE</param-name>
    <param-value>java:comp/env/jdbc/default</param-value>
</context-param>

<context-param>
    <param-name>LIQUIBASE_HOST_EXCLUDES</param-name>
    <param-value>production1.example.com, production2.example.com</param-value>
</context-param>

<context-param>
    <param-name>LIQUIBASE_FAIL_ON_ERROR</param-name>
    <param-value>true</param-value>
</context-param>

<context-param>
    <param-name>LIQUIBASE_CONTEXTS</param-name>
    <param-value>production</param-value>
</context-param>

<listener>
    <listener-class>liquibase.servlet.LiquibaseServletListener</listener-class>
</listener>

```

#### 2.29.1 `context-param`

| 参数                      | 1.9版本                   | 描述                                                         |
| ------------------------- | ------------------------- | ------------------------------------------------------------ |
| `liquibase.changelog`     | `LIQUIBASE_CHANGELOG`     | 指定要运行的`changelog`文件。**必填**                        |
| `liquibase.datasource`    | `LIQUIBASE_DATA_SOURCE`   | 为了运行`Liquibase`使用`JNDI`数据源。请注意，如果 `LIQUIBASE_DATA_SOURCE`没有足够的权限创建/更改表等，则 `LIQUIBASE_DATA_SOURCE` 可能不同于 `Web` 应用的其余部分使用的数据源。**必填** |
| `liquibase.host.excludes` | `LIQUIBASE_HOST_EXCLUDES` | 指定不希望 `Liquibase` 运行的主机名。指定此参数允许您将相同的 `WAR/EAR` 部署到不同环境中的多台计算机，并且不会在所有计算机上运行 `Liquibase`。 |
| `liquibase.host.includes` | `LIQUIBASE_HOST_INCLUDES` | 指定仅希望 `Liquibase` 运行的主机名。指定此参数允许您将相同的 `WAR/EAR`部署到不同环境中的多台计算机，并且不会在所有计算机上运行`Liquibase`。 |
| `liquibase.onerror.fail`  | `LIQUIBASE_FAIL_ON_ERROR` | 指定在发生错误时，`Liquibase`是否引发异常。将值设置为`true`（默认值）将导致引发异常，并阻止网站正确初始化。将值设置为`false`将允许站点正常部署，但数据库将处于未定义状态。 |
| `liquibase.contexts`      | `LIQUIBASE_CONTEXTS`      | 要运行的上下文列表，用逗号分隔开                             |

如果要控制运行 `Liquibase` 但不想设置 `LIQUIBASE_HOST_EXCLUDES`/`LIQUIBASE_HOST_INCLUDES` 属性的服务器，则可以指定`liquibase.should.run=[true/false]`系统属性。

### 2.30 集成`Spring`

`Liquibase`可以通过声明一个`liquibase.spring.SpringLiquibase`的`bean`在`spring`的环境中运行。

#### 2.30.1 例子

```xml
<bean id="liquibase" class="liquibase.integration.spring.SpringLiquibase">
      <property name="dataSource" ref="myDataSource" />
      <property name="changeLog" value="classpath:db-changelog.xml" />

      
      <property name="contexts" value="test, production" />
 </bean>

```

#### 2.30.2 可用的属性

*   `beanName`
*   `changeLog`
*   `contexts`
*   `dataSource`
*   `defaultSchema`
*   `dropFirst` **从版本2.0.2**
*   `parameters`
*   `resourceLoader`

* * *

### 2.31 离线数据库支持

如果您无法直接针对数据库运行 `Liquibase`，则有两种主要选项可确保数据库保持最新。

#### 2.31.2 `updateSQL`

我的第一个回答是"你真的需要简化它吗？您构建了很长一段时间的`changelog`，并且已运行它并测试了无数次。一旦你开始乱搞的`changelog`文件，你引入的风险有它自己的成本。无论您有什么性能或文件大小问题，您是否真的会超过干扰您知道有效的脚本的风险？

如果值得冒险，为什么它起作用的风险？有时的问题是，您的 `changelog` 文件变得太大，以至于编辑器会窒息，或者您得到太多的合并冲突。最好的方法是简单地将`changelog`文件分解为多个文件。创建 `master.changelog.xml`文件，该文件使用 标记引用其他`changelog`文件，而不是让单个 `changelog.xml`文件包含所有内容。

```xml
<databaseChangeLog
            xmlns="http://www.liquibase.org/xml/ns/dbchangelog/3.3"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/3.3
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.3.xsd">
    <include file="com/example/news/news.changelog.xml"/>
    <include file="com/example/directory/directory.changelog.xml"/>
</databaseChangeLog>

```

当您对 `master.changelog.xml` 文件运行 `liquibase` 更新时，将运行 `com/example/news/news.changelog.xml`中的`changesets`，然后运行`com/example/directory/directory.changelog.xml` 中的`change sets`。您可以以最适合您的任何方式分解`changesets`。有些按功能分解，有的按发布将其分解。找出最适合你的。

其他时候，问题是`liquibase`更新需要的时间太长。`Liquibase` 在将 `DATBASECHANGELOG` 表的内容与当前`changelogs`文件进行比较时，尽量提高效率，即使有数千个已运行的`changeset`，运行`update`命令也只需几秒钟即可运行。如果您发现更新所花的时间比它应该要长，请查看 `Liquibase`日志以确定原因。也许是因为设置了`runAlways="true"`，所以运行过的也会运行。`changeSet`不再需要运行或有不再需要的`preConditions`。使用 `-logLevel=INFO`或者`-logLevel=DEBUG` 运行 `Liquibase` 可以提供额外的输出，可帮助您确定哪些`changeSets`速度较慢。一旦您知道什么正在减慢更新速度，请尝试仅更改这些`changeSets`，而不是放弃整个`changeLog`并从头开始。您仍希望深入测试`changelog`，但这是一个风险要小得多的更改。

对于其他人，他们发现 `liquibase`更新非常适合增量更新，但从头开始创建数据库需要很长时间。我再次问"这真的是个问题吗？是否经常重新创建数据库，以便更改创建脚本的风险有意义？如果是，第一步应该是查找如上所述的问题`changeSet`。数据库速度很快，尤其是当它们是空的。即使您创建表只是为了再次删除它，通常也只是几毫秒的开销，不值得优化。**创建数据库的最大性能瓶颈通常是索引**，因此请从索引开始。如果在创建过程中频繁创建和更新索引，则可以将这些`changeSet`并到更高效的内容中。

当您需要`surgically`地更改现有`changeSets`时，请记住 `Liquibase` 的工作原理：每个`change Set` 都有一个`id`、一个`author`和一个`filepath`，这些路径共同唯一地标识它。如果 `DATABASECHANGELOG` 表具有该更改的条目设置，则不会运行它。如果它有一个条目，则如果文件中的 `changeSet` 的校验和与上次运行时存储的内容不匹配，则它将引发错误。

如何修改现有`changeSet`还取决于您的环境以及问题`changeSet`在`changelog`中的位置。如果要修改已应用于所有环境且现在仅用于新数据库生成上的`chaneSet`，则可以以不同于已应用于某些数据库但尚未应用于其他数据库的`changeSet`。

要合并或修改现有`changeSet`，您需要执行编辑现有`changeSet`、删除旧`changeSet`和创建新`changeSet`的组合

删除不需要的`changeSet`很容易，因为`Liquibase` 不关心没有相应`changeSet`的 `DATABASEchangeLOG`行。只需删除过期的`changeSet`，您就完成了。例如，如果您有一个创建表`cart`的`changeSet`，而另一个`changeSet`则删除文件上的两个`changeSet`。但是，必须确保使用该表的创建和删除之间没有`changeSet`，否则它们将在新的数据库生成中失败。这是更改`changelog`文件时引入风险的示例:

相反，假设您有一个`cart`表，该表是在一个 `changeSet` 中创建的，然后在另一个更改组中创建`promo_code`列，在另一个更改中创建`abandoned`标志。

```xml
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                       xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.3.xsd">
    <changeSet author="nvoxland" id="1">
        <createTable tableName="cart">
            <column name="id" type="int"/>
        </createTable>
    </changeSet>

    <changeSet author="nvoxland" id="2">
        <addColumn tableName="cart">
            <column name="promo_code" type="varchar(10)"/>
        </addColumn>
    </changeSet>

    <changeSet author="nvoxland" id="3">
        <addColumn tableName="cart">
            <column name="abandoned" type="boolean"/>
        </addColumn>
    </changeSet>
</databaseChangeLog>

```

一个选项是使用现有`id="1"` 将所有内容合并到新的`changeSet`中，然后删除其他`changeSet`

```xml
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.3.xsd">
    <changeSet author="nvoxland" id="1">
        <validCheckSum>7:f24b25ba0fea451728ffbade634f791d</validCheckSum>
        <createTable tableName="cart">
            <column name="id" type="int"/>
            <column name="promo_code" type="varchar(10)"/>
            <column name="abandoned" type="boolean"/>
        </createTable>
    </changeSet>
</databaseChangeLog>

```

如果所有现有数据库都具有具有已添加的`promo_code`和已添加的`abandon`列的`cart`表，则此方法将正常工作。针对现有数据库运行 `Liquibase` 只会看到 `id="1"`已运行，并且不会执行任何新操作。在空白数据库上运行 `Liquibase` 会创建包含所有列的`cart`表。请注意，我们必须添加标志或现有数据库将出现错误，指出 `id="1"`自运行以来已更改。只需使用`vaildCheckSum` 标签中的错误消息中的校验和，以标记您知道它已更改且新值正常。

如果您有一些尚未添加`promo_code`和/或`abandoned`列的数据库，请像以前那样更新原始创建表，但使用 `onFail="MARK_RAN"`的`preConditions`来处理旧`changeSet`运行而仍未添加列的情况如果新`changeSet`运行。

```xml
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.3.xsd">
    <changeSet author="nvoxland" id="1">
        <validCheckSum>7:f24b25ba0fea451728ffbade634f791d</validCheckSum>
        <createTable tableName="cart">
            <column name="id" type="int"/>
            <column name="promo_code" type="varchar(10)"/>
            <column name="abandoned" type="boolean"/>
        </createTable>
    </changeSet>

    <changeSet author="nvoxland" id="2">
        <preConditions onFail="MARK_RAN">
            <not><columnExists tableName="cart" columnName="promo_code"/></not>
        </preConditions>
        <addColumn tableName="cart">
            <column name="promo_code" type="varchar(10)"/>
        </addColumn>
    </changeSet>

    <changeSet author="nvoxland" id="3">
        <preConditions onFail="MARK_RAN">
            <not><columnExists tableName="cart" columnName="abandoned"/></not>
        </preConditions>
        <addColumn tableName="cart">
            <column name="abandoned" type="boolean"/>
        </addColumn>
    </changeSet>

</databaseChangeLog>

```

现在，在现有数据库上已运行所有 3 个`changeSet`，`Liquibase` 将一如既往地继续运行。对于具有旧`cart`定义的现有数据库，它将看到 `id="2"`和 `id="3"`的列不存在，然后照常执行。对于空白数据库，它将使用 `promo_code` 和`abandoned`列创建表，然后在 `id="2"`和`id="3"`中，它将看到它们已在那里，并标记它们已运行而不重新添加列。但是，警告：使用`preConditions`会增加更新执行的性能开销，并在`updateSQL` 模式下被忽略，因为 `Liquibase` 无法知道在`changesets`尚未实际执行时它们有多适用。因此，最好尽可能避免使用，但一定要在需要时使用它们。`preConditions`还会增加`changeLog`的复杂性，这将需要额外的测试，因此在决定是否修改`changeLog`逻辑时请记住这一点。有时，最好、最安全的方法是等到所有数据库都拥有列，然后修改`changeSet`以避免`preConditions`。

`cart/promo_code/abandoned`示例显示了在修改现有`changeSet`时可以使用的一些基本模式。类似的`patters`可用于优化您的瓶颈。请记住，当您更改一个`changeSet`时，它可能会影响下面的其他`changeSets`，这些更改可能也需要修改。这很容易失控，所以要注意你在做什么。

如果您最终发现完全重新启动`changelog`最合算，请参阅["将 `Liquibase` 引入现有项目"](https://www.liquibase.org/documentation/existing_project.html)，其中描述了如何将 `Liquibase`添加到现有项目（即使该项目以前由 `Liquibase` 管理）。

### 2.32 用户和开发人员社区

**关于如何使用`Liquibase`的疑问?**

使用`liquibase`标签可以在`StackOverFlow`上面找到基本多数关于`Liquibase`使用的问题http://stackoverflow.com/tags/liquibase.

**有新的功能想法？你在`Liquibase`上面发现了一个`BUG`?**

如果在`Liquibase`的库中发现了`BUG`,请在[Liquibase Jira](http://liquibase.jira.com/browse/CORE).上报告它。

如果是带有扩展的 `Bug`，请记录它扩展的 `github` 页面部分. 例如 https://github.com/liquibase/liquibase-hibernate/issues或 https://github.com/liquibase/liquibase-oracle/issues.

所有 `Liquibase` 管理的扩展都可以在 [Liquibase GitHub group](https://github.com/liquibase)上找到

**正在寻找`Liquibase`的最新消息吗？**

`Liquibase`发布公告以及一般项目新闻和文章交叉张贴到`Liquibase`博客http://www.liquibase.org/blog和邮件列表。

要订阅邮件列表，请使用左侧的"通知"表单。[@liquibase](https://twitter.com/liquibase)

**正在找`Liquibase`的源码吗?**

`Liquibase` 源码挂在 https://github.com/liquibase/liquibase.代码提交应经过标准的 `GitHub`拉取请求系统。

获取更多的信息，可以参考 [开发文档](http://www.liquibase.org/development/index.html)

**查看超出`stackOverFlow`范围的讨论?**

*   [Liquibase Users](http://forum.liquibase.org/#Forum/liquibase-users) - 用于使用 `Liquibase`和编写扩展名的主题。
*   [Liquibase Developers](http://forum.liquibase.org/#Forum/liquibase-development) -与 `Liquibase`代码库工作相关的主题。

**需要直接到顶部?**

你总是可以联系`Nathan Voxland`,致力于 `Liquibase`的开源。 nathan.voxland@liquibase.org 或者[@nvoxland](https://twitter.com/nvoxland)

**寻求支付支持?**

商业培训和支持可从 [datical.com](http://www.datical.com/liquibase/)上获取。

### 2.33 数据库的`"diff"`

然而最好跟踪数据库变化的方式是在开发中增加`changesets`。(可以参考 [the problem with database diffs](http://www.liquibase.org/2007/06/the-problem-with-database-diffs.html)), 有时能够执行数据库差异是有价值的，尤其是在项目即将结束时，作为双重检查，所有必需的更改都包含在`changelog`中。

#### 2.33.1 运行差异

`Diff`命令支持通过 [command\_line](https://www.liquibase.org/documentation/command_line.html) 和 [ant](https://www.liquibase.org/documentation/ant/index.html) 两种工具获得。当`diff-ing`数据库的时候， 像在 `Liquibase` 中通常那样指定目标数据库（`-url`、`-username`等标志），并在命令名称后使用附加标志指定基本数据库

例子

```yaml
liquibase.sh --driver=oracle.jdbc.OracleDriver \
        --url=jdbc:oracle:thin:@testdb:1521:test \
        --username=bob \
        --password=bob \
    diff \
        --referenceUrl=jdbc:oracle:thin:@localhost/XE \
        --referenceUsername=bob \
        --referencePassword=bob

```

#### 2.33.2 数据库比较

目前，`Liquibase` 运行以下比较：

*   版本差异
*   丢失/不期望的表
*   丢失/不期望的视图
*   丢失/不期望的列
*   丢失/不期望的主键
*   丢失/不期望的唯一约束
*   丢失/不期望的外键
*   丢失/不期望的序列
*   丢失/不期望的索引
*   列定义差异（数据类型，自动增长等）
*   视图定义差异
*   数据差异 (限制), 默认是 不校验的

（当前）是不校验的：

*   非外键约束（检查等）
*   存储过程
*   数据类型长度

`Liquibase` 可以区分不同的数据库类型，但由于大小写和数据类型的差异，结果可能会偏差。

#### 2.33.3 控制检查（自 1.8 起）

可以使用`diff`命令的 `diffType` 参数控制检查的更改。以下选项可用，可以作为逗号分隔的列表传递：

*   `tables` **\[默认\]**

*   `columns`**\[默认\]**
*   `views` **\[默认\]**
*   `primaryKeys` **\[默认\]**
*   `indexes` **\[默认\]**
*   `foreignKeys` **\[默认\]**
*   `sequences` **\[默认\]**
*   `data`

如果未指定差异类型，将运行标记为 `DEFAULT` 的检查。

注意: 这仅仅作用于`generateChangeLog`命令, 不会作用于`diff` 或者 `diffChangeLog`命令。

#### 2.33.4 输出模式

`Liquibase` 支持两种输出模式：`report`模式（`diff`）和`changelog`模式（`diffChangeLog`）。在这两种模式下，`diff`进度在执行过程中都会报告为标准错误。

#### 2.33.5 报告模式

在报告模式下，两个数据库之间的差异描述将报告并标准输出。

```yaml
Base Database: BOB jdbc:oracle:thin:@testdb:1521:latest
Target Database: BOB jdbc:oracle:thin:@localhost/XE
Product Name: EQUAL
Product Version:
     Base:   'Oracle Database 10g Enterprise Edition Release 10.2.0.1.0
With the Partitioning, OLAP and Data Mining options'
     Target: 'Oracle Database 10g Express Edition Release 10.2.0.1.0'
Missing Tables: NONE
Unexpected Tables: NONE
Missing Views: NONE
Unexpected Views: NONE
Missing Columns:
     CREDIT.MONTH
     CREDIT.COMPANY
     CMS_TEMPLATE.CLASSTYPE
     CONTENTITEM.SORTORDER
Unexpected Columns:
     CATEGORY.SORTORDER
Missing Foreign Keys: NONE
Unexpected Foreign Keys:
     FK_NAME (ID_VC -> STATUS_ID_VC)
Missing Primary Keys: NONE
Unexpected Primary Keys: NONE
Missing Indexes: NONE
Unexpected Indexes: NONE
Missing Sequences: NONE
Unexpected Sequences: NONE

```

#### 2.33.6 `changelog`模式

在`changeLog`模式下，将基本数据库升级到目标数据库所需的 `XML` `changeLog`发送标准输出。此`changeLog`可以原样包含，也可以复制到现有`changeLog`中。如果 `diff` 命令传递了现有的`changelog`文件，则新的`changeSets`将追加到文件末尾。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.1"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.1
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.1.xsd">
    <changeSet author="diff-generated" id="1185206820975-1">
        <addColumn tableName="CREDIT">
            <column name="MONTH" type="VARCHAR2(10)"/>
        </addColumn>
    </changeSet>
    <changeSet author="diff-generated" id="1185206820975-2">
        <addColumn tableName="CREDIT">
            <column name="COMPANY" type="NUMBER(22,0)"/>
        </addColumn>
    </changeSet>
    <changeSet author="diff-generated" id="1185206820975-3">
        <addColumn tableName="CMS_TEMPLATE">
            <column name="CLASSTYPE" type="VARCHAR2(255)"/>
        </addColumn>
    </changeSet>
    <changeSet author="diff-generated" id="1185206820975-4">
        <addColumn tableName="CONTENTITEM">
            <column name="SORTORDER" type="NUMBER(22)"/>
        </addColumn>
    </changeSet>
    <changeSet author="diff-generated" id="1185206820975-5">
        <dropColumn columnName="SORTORDER" tableName="CATEGORY"/>
    </changeSet>
    <changeSet author="diff-generated" id="1185206820975-6">
        <dropForeignKeyConstraint baseTableName="CMS_STATUS"
                     constraintName="FK_NAME"/>
    </changeSet>
</databaseChangeLog>

```

可以使用`includeObjects`或`excludeObjects`参数控制要包括在`changelog`中的数据库对象。（自 3.3.2 起）

支持的格式为:

*   对象名称（实际上是 `regexp`）将匹配其名称与 `regexp`匹配的任何对象。
*   类型：与给定类型对象的 `regexp`名称匹配的名称语法。
*   如果需要多个表达式，逗号将它们分开
*   类型：命名逻辑将应用于包含列、索引等的表。

注：名称比较区分大小写。如果需要不敏感逻辑，请使用 "（?i）"正则标志。

过滤例子：

*   `table_name`只会匹配一个叫`table_name`的表，不会匹配叫`other_table`或者 `TABLE_NAME`的表
*   可以匹配`table_name` 和 `TABLE_NAME`
*   在表`table_name`中的`"table_name"`会匹配所有的列。
*   `table:table_name`将会匹配一张叫做`table_name`的表，而不会匹配一个列叫`table_name`的表
*   `table:table_name,columns:*.lock`将会匹配一个叫做`table_name`的表，并且所有的列都是以`lock`结尾的。

* * *

### 2.34 在已经存在的项目中添加`Liquibase`

[快速入门指南](https://blog.csdn.net/u012934325/article/details/https%EF%BC%9A//www.liquibase.org/quickstart.html)非常适合启动`Liquibase`新项目，因为您的`changelog`文件与您的数据库都是空的，很容易匹配。但是，当您有一个现有项目与现有数据库时，事情会更加复杂。

不幸的是，没有简单的"这是你如何做"的答案，因为有这么多的变化的项目，流程和要求。`Liquibase`提供了许多工具来帮助该过程，但由您决定针对您特定情况组合它们的最佳方式。

将 `Liquibase` 添加到现有项目时，基本上有两种方法：让它看起来像您一直在使用 `Liquibase`和开始使用 `Liquibase`

让它看起来像您一直在使用 \`Liquibase

The goal of this approach is to have a changelog file that matches the current state of your database. You can run this changeLog against a blank database and the final result will be indistinguishable from your existing databases–as if you used Liquibase from the beginning. This approach is usually the best long term, but it can be more work up front.

此方法的目标是具有与数据库的当前状态相匹配的`changelog`文件。您可以针对空数据库运行此`changelog`，最终结果与现有数据库无法区分，就像从一开始就使用 `Liquibase` 一样。

创建`changelog`

Creating the changelog to match your database can be done automatically using the [generateChangeLog command](https://www.liquibase.org/documentation/generating_changelogs.html) or be done manually. For any database larger than a few tables, the generateChangeLog command is usually a good idea but make sure you go through the generated changeSets to ensure they are correct. Liquibase does not always detect more complex structures like stored procedures or details like if an index is not clustered. Also, ensure data types are as you expected them.

创建与数据库匹配的`changelog`可以使用 [`generateChangeLog` 命令](https://www.liquibase.org/documentation/generating_changelogs.html)自动完成，也可以手动完成。对于任何大于几个表的数据库，生成`ChangeLog`命令通常是个好主意，但请确保通过生成的`changesets`，以确保它们是正确的。`Liquibase` 并不总是检测更复杂的结构，如存储过程或详细信息，例如如果索引未群集。此外，请确保数据类型与预期相同。

### 2.35 Populate the DatabaseChangeLog table

Once you have your changeLog, you need a way to ensure that the pre-Liquibase changeSets are only ran on new, empty databases. The easiest way to do this is generally to use the `changeLogSync` or `changeLogSyncSQL` command to execute (or generate) the SQL that marks the starting changeSets as already ran without actually executing them.

As an alternative to the changeLogSync command, you can add [contexts](https://www.liquibase.org/documentation/contexts.html) on the pre-Liquibase changeSets such as `<changeSet ... context="legacy">` and when you run Liquibase on a new database you run with `liquibase --contexts=legacy update` and on an existing database you run with `liquibase --contexts=non-legacy`.

Finally, you can add `<precondition onFail="MARK_RAN">` tags to the generated changeSets. For example, if you have a `<createTable tableName="person">` changeSet, you would add `<preconditions onFail="MARK_RAN"><not><tableExists tableName="person"/></not></preconditions>` tag. Adding preconditions requires more changes to the changeLog file and introduces a performance penalty because Liquibase must check the database metadata for each changeSet the first run through, this approach is usually best used in isolated cases only.

### 2.36 What is the current state?

Often times a part of the reason to move to Liquibase is because your schemas have diverged over time, so an important question to answer is “If I’m making the changelog file match the current state, what **is** the current state?” Usually the best answer to that question is “the production database” but it can vary.

How divergent your schemas are will also affect which of the above techniques you use to populate the DatabaseChangeLog table, and it will often times make sense to use multiple approaches. For example, you may want to generate your base changeLogs from the production database and use changeLogSyncSQL to be able to mark them ran on everything from production down. Then you can add your non-released changeSets to the changeLog file with a precondition checking if it has already ran. That will allow Liquibase to automatically figure out the correct state for all your databases from development through production.

3. We are going to use Liquibase starting……NOW!
--------------------------------------------

Instead of building up a changeLog to match your existing database, you can instead just declare “from now on we are using Liquibase”. The advantage to this is that it much easier to set up because it is just a mandate. Usually this works best going from one version to the next because your databases are all in a reasonably consistent state and you simply start tracking database changes in your next version using Liquibase. Because Liquibase only looks at the DatabaseChangeLog table to determine what needs to run, it doesn’t care what else might be in your database and so it will leave all your existing tables alone and just run the new changeSets.

The biggest disadvantage to this approach is that you cannot bootstrap an empty database with Liquibase alone. A work-around is to take a pre-Liquibase snapshot using your database backup tool and use that as your database seed. Any time you need to create a new database, you first load in the seed and then run Liquibase update.

Depending on how much variation you have between your schemas, even with this approach you may need to rely on preconditions or a “mark changes ran” script in order to standardize and handle those variations.

4. People and Processes
--------------------

Finally, remember that starting to use Liquibase–especially on an existing project–isn’t just about how you bootstrap your changeLog file. It is also a question of how you introduce Liquibase into your existing processes and culture.

For many companies and projects, everyone realizes the problems that need fixing and are on board with the advantages of change. For others, however, there can be entrenched interests and strong resistance similar to any other process change. Liquibase provides many tools and approaches that can be used to ease it into an existing process such as [SQL output](https://www.liquibase.org/documentation/sql_output.html), [SQL formatted changelogs](https://www.liquibase.org/documentation/sql_format.html), [diffChangeLog](https://www.liquibase.org/documentation/diff.html) and more that can be combined in ways that works best for your group.

If you know that introducing Liquibase is going to be complex, either from a technical or processes standpoint, it is usually best to introduce it slowly. Start with it on a new project as a trial run and once you have a good grasp of how it works and available options, apply it to other existing projects.

* * *

Due to variations in data types and SQL syntax, the following databases are currently supported out of the box. Additional databases as well as enhancements to support for the below databases are available through [Liquibase extensions](http://liquibase.org/extensions)

Please find further information about which JDBC driver, URL, classes etc. these databases need, by clicking on the database-specific links in the table below.

| Database                                                     | Type Name  | Notes                                                        |
| ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| MySQL                                                        | mysql      | No Issues                                                    |
| PostgreSQL                                                   | postgresql | 8.2+ is required to use the “drop all database objects” functionality. |
| Oracle                                                       | oracle     | 11g driver is required when using the diff tool on databases running with AL32UTF8 or AL16UTF16 |
| Sql Server                                                   | mssql      | No Issues                                                    |
| Sybase\_Enterprise                                           | sybase     | ASE 12.0+ required. “select into” database option needs to be set. Best driver is JTDS. Sybase does not support transactions for DDL so rollbacks will not work on failures. Foreign keys can not be dropped which can break the rollback or dropAll functionality. |
| Sybase\_Anywhere                                             | asany      | **Since 1.9**                                                |
| DB2                                                          | db2        | No Issues. Will auto-call REORG when necessary.              |
| [Apache\_Derby](https://www.liquibase.org/apache_derby.html) | derby      | No Issues                                                    |
| HSQL                                                         | hsqldb     | No Issues                                                    |
| H2                                                           | h2         | No Issues                                                    |
| [Informix](https://www.liquibase.org/informix.html)          | informix   | No Issues                                                    |
| Firebird                                                     | firebird   | No Issues                                                    |
| [SQLite](https://www.liquibase.org/sqlite.html)              | sqlite     | No Issues                                                    |

As of Liquibase v3.1, support for some less common databases has been moved out of Liquibase core and into extensions.

To re-enable support for these databases, install the corresponding extension:

*   [InterSystems Cache](https://github.com/liquibase/liquibase-cache)
*   [SAP MaxDB](https://github.com/liquibase/liquibase-maxdb)
*   [IBM DB2 for iSeries](https://github.com/liquibase/liquibase-db2i)

Since Liquibase is built on top of standard JDBC, the only ties it has to the underlying database is through the SQL that can vary from DBMS to DBMS. If you attempt to use Liquibase with an unsupported database, it will try to run and will most likely succeed. The only problem you are likely to run into is the current date/time function name. If Liquibase is unable to determine the correct date/time function, you can pass it in via the [“command line”](https://www.liquibase.org/documentation/command_line.html) and [documentation/Ant](https://www.liquibase.org/documentation/ant/index.html)).

You may also run into problem with the SQL generated by the change/refactoring tags on unsupported databases. The best way to deal with this problem is to first try the standard change/refactoring tags. If it generates an error, you can fall back to the [sql change](https://www.liquibase.org/documentation/changes/sql.html) to code whatever change you need to make in a way that your database understands.

If, for some reason, the DatabaseChangeLog table cannot be created on your database, the base creation SQL that you can modify to suit your needs is:

```
CREATE TABLE DATABASECHANGELOG (id varchar(150) not null,
author varchar(150) not null,
filename varchar(255) not null,
dateExecuted datetime not null,
md5sum varchar(32),
description varchar(255),
comments varchar(255),
tag varchar(255),
liquibase varchar(10),
primary key(id, author, filename))

```

Reasons for creating the DatabaseChangeLog table yourself include database that require null fields to be specified as such and index limitations that don’t allow primary keys on fields as long. You can change the data types and or data type lengths all you want.

### 4.1 `Liquibase Java Doc API`

https://www.liquibase.org/javadoc/index.html

### 4.2 `Liquibase`的`maven`插件

https://www.liquibase.org/documentation/maven/index.html

> 感谢网友翠星帮我做校对

