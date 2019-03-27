---
title: Kettle插件架构
date: 2017-04-15 19:42:49
tags: [开源项目,kettle]
categories: [开源项目,kettle]
---
#  #Kettle插件体系
最近公司内有业务系统到数据中心同步的升级改造需求，从各个业务系统收集增量数据到数据中心的数据仓库平台。因为开发周期短暂，需要快速的响应，开发出可用的产品，所以决定借鉴开源程序Kettle，开发一个文件解析组件，然后利用Kettle平台的大数据组件进行与数据中心大数据平台对接

数据同步部分是：业务系统（RDBMS）->Kettle(azkaban进行调度)->数据中心，因为Kettle的增量抽取组件经常出现数据不一致等问题，所以目前已更改为：业务系统（RDBMS）->OGG（CDC增量抽取）->数据中心的方式。

本文主要介绍如何扩展Kettle的功能，部分内容来自《Pentaho Kettle解决方案：使用PDI构建开源ETL解决方案》一书，推荐购买阅读。
##  ##架构
我们先看Kettle插件架构。
 ![](http://i.imgur.com/mLvXMuV.jpg)
从功能上看，Kettle内部的对象和外部插件没有任何区别。因为它们使用的API都是一样的，它们只是在运行时的加载方式不同。
从Kettle4以后，Kettle内部有一个插件注册系统，它负责加载各种内部和外部插件。插件有以下两个标识属性。
**插件类型**：由PluginTypeInterface接口定义。例如StepPluginType、JobEntryPluginType、PartitionerPluginType和RepositoryPluginType。
**插件ID**：这是一个字符串数组，用来唯一标识一个插件。因为旧的插件可以被新的插件代替，一个插件可以有多个ID。在大多数情况下，插件只使用一个单一的字符串，如TableInput是“表输入”步骤的ID，MYSQL是MySQL数据库类型的ID。
当Kettle环境初始化以后，插件注册系统首先加载所有的内部对象，Kettle读取下面的配置文件来加载内部对象，这些配置文件位于Kettle的.jar文件中。
	 Kettle-steps.xml：内部转换步骤。
	 Kettle-job-entries.xml：内部作业项。
	 Kettle-partition-plugins.xml：内部分区类型。
	 Kettle-database-types.xml：内部数据库类型。
	 Kettle-repositories.xml：内部资源库类型。

插件注册系统加载了所有的内部对象后，就要搜索可用的外部插件。通过浏览plugins/目录的各个子目录下的.jar文件来完成。它搜索特定的Kettle annotations来判断一个类是否是插件。加载过程将在本章的后面介绍。
因为在内部对象加载后才加载插件，所以插件会替代相同ID的已加载的内部对象。例如，你创建了插件，插件的ID是TableInput，就可以替换Kettle标准的“表输入”步骤。这个功能可以让你用插件替换Kettle内置的步骤。可以通过子类继承方式，直接扩展已有步骤的某些功能。

##  ##插件类型
Kettle有下面几种插件类型（下面的插件是Kettle4.0的插件类型，新版kettle包含了很多新的插件，比如视图插件、大数据插件等等）。

- 转换步骤插件：在Kettle转换中使用的步骤，用来处理数据行。


- 作业项插件：在Kettle作业中使用的作业项，用来实现某个任务。
	

- 分区方法插件：利用输入字段的值指定自己的分区规则。
	

- 数据库类型插件：用来扩展不同的数据库类型。
	

- 资源库类型插件：可以把Kettle元数据保存为自定义类型或格式。

说明：除了这些类型，还有Spoon类型的插件，可以把功能扩展到Spoon，本书不介绍这个功能。
##  ##转换步骤插件
转换步骤插件包括了四个Java类，这四个类分别实现四个接口。


- StepMetaInterface：这个接口对外 提供步骤的元数据并处理串行化。


- StepInterface:这个接口根据上面接口提供的元数据，来实现步骤的具体功能。


- StepDataInterface:这个接口用来存储步骤的临时数据、文件句柄等。


- StepDialogInterface:这个接口是Spoon里的图形界面，用来编辑步骤的元数据。

接下来，我们介绍这些接口的基本内容。对于每个接口，在一个简单的“Hello World”例子里提供这些类的相应实现。“Hello World”例子将在数据流里增加一个字段，字段名用户可以自定义，字段值是”Hello world!“。最后介绍一下如何部署这个例子。
###  ###StepMetaInterface
接口org.pentaho.di.trans.step.StepMetaInterface负责步骤里所有和元数据相关的任务。和元数据相关的工作包括：  
元数据和XML(或资源库)之间的序列化和反序列化  
getXML（）和loadXML()  
saveRep()和readRep()  

描述输出字段  
getFields()  

检验元数据是否正确  
Check()  

获取步骤相应的要SQL语句，使步骤可以正确运行  
getSQLStatements()  

给元数据设置默认值  
setDefault()  

完成对数据库的影响分析  
analyseImpact()  

描述各类输入和输出流  
getStepIOMeta()  
searchInfoAndTargetSteps()  
handleStreamSelection()  
getOptionalStreams()  
resetStepIoMeta()  

导出元数据资源  
exportResources()  
getResourceDependencies()  

描述使用的库  
getUsedLibraries()  

描述使用的数据库连接  
getUsedDatabaseConnections()  

描述这个步骤需要的字段（通常是一个数据库表）  
getRequiredFields()  

描述步骤是否具有某些功能  
supportsErrorHandling()  
excludeFromRowLayoutVerification()  
excludeFromCopyDistributeVerification()  

这个接口里还定义了几个方法来说明这四个接口如何结合到一起。  
String getDialogClassName():用来描述实现了StepDialogInterface接口的对话框类的名字。如果这个方法返回了null，调用类会根据实现了StepMetaInterface接口的类的类名和包名来自动生成对话框类的名字。  
SetpInterface getStep():创建一个实现了StepInterface接口的类。  
StepDataInterface getStepData():创建一个实现了StepDataInterface接口的类。  
现在我们看看”Hello World”例子里对SetpMetaInterface接口的实现  
HelloworldStepMeta.java
    package org.kettlesolutions.plugin.step.helloworld;
    
    import java.util.List;
    import java.util.Map;
    
    import org.pentaho.di.core.CheckResult;
    import org.pentaho.di.core.CheckResultInterface;
    import org.pentaho.di.core.Const;
    import org.pentaho.di.core.Counter;
    import org.pentaho.di.core.annotations.Step;
    import org.pentaho.di.core.database.DatabaseMeta;
    import org.pentaho.di.core.exception.KettleException;
    import org.pentaho.di.core.exception.KettleStepException;
    import org.pentaho.di.core.exception.KettleXMLException;
    import org.pentaho.di.core.row.RowMetaInterface;
    import org.pentaho.di.core.row.ValueMeta;
    import org.pentaho.di.core.row.ValueMetaInterface;
    import org.pentaho.di.core.variables.VariableSpace;
    import org.pentaho.di.core.xml.XMLHandler;
    import org.pentaho.di.i18n.BaseMessages;
    import org.pentaho.di.repository.ObjectId;
    import org.pentaho.di.repository.Repository;
    import org.pentaho.di.trans.Trans;
    import org.pentaho.di.trans.TransMeta;
    import org.pentaho.di.trans.step.BaseStepMeta;
    import org.pentaho.di.trans.step.StepDataInterface;
    import org.pentaho.di.trans.step.StepInterface;
    import org.pentaho.di.trans.step.StepMeta;
    import org.pentaho.di.trans.step.StepMetaInterface;
    import org.w3c.dom.Node;
    
    @Step(
    		id="Helloworld",
    		name="name",
    		description="description",
    		categoryDescription="categoryDescription", 
    		image="org/kettlesolutions/plugin/step/helloworld/HelloWorld.png",
    		i18nPackageName="org.kettlesolutions.plugin.step.helloworld"
    ) 
    public class HelloworldStepMeta extends BaseStepMeta implements StepMetaInterface {
    	/**
    	 * PKG变量说明了messages包的位置，在messages包里有各种国际化的资源文件。
    	 * 在本章后面经常要看到的BaseMessages.getString()方法，就是根据软件的国际化
    	 * 设置，从不同的文件中获取文字。PKG变量通常位于类的最上方，被国际化图形工具使用，
    	 * 通过国际化图形工具，国际化人员可以编辑不同的国际化资源文件。所以我们会在很多Kettle
    	 * 代码里看见这样的结构。
    	 */
    	private static Class<?> PKG = HelloworldStep.class; //for i18n
    	public enum Tag {//field_name用于保存用户输入的字段名：保存“Hello，world！"字符串的字段名。
    		field_name,
    	};
    	
    	private String fieldName;
    	
    	/**
    	 * @return the fieldName
    	 */
    	public String getFieldName() {
    		return fieldName;
    	}
    
    	/**
    	 * @param fieldName the fieldName to set
    	 */
    	public void setFieldName(String fieldName) {
    		this.fieldName = fieldName;
    	}
    
    	/**
    	 * checks parameters, adds result to List<CheckResultInterface>
    	 * used in Action > Verify transformation
    	 * 验证用户是否在对话框里输入了字段名，并把验证结果添加到检验转换时出现的问题列表里。（最好
    	 * 要检验用户输入的所有选项，而不只是容易出错的选项）
    	 */
    	public void check(List<CheckResultInterface> remarks, TransMeta transMeta, StepMeta stepMeta, 
    			RowMetaInterface prev, String input[], String output[], RowMetaInterface info) {
    		
    		if (Const.isEmpty(fieldName)) {
    			CheckResultInterface error = new CheckResult(
    				CheckResult.TYPE_RESULT_ERROR, 
    				BaseMessages.getString(PKG, "HelloworldMeta.CHECK_ERR_NO_FIELD"), 
    				stepMeta
    			);
    			remarks.add(error);
    		} else {
    			CheckResultInterface ok = new CheckResult(
    				CheckResult.TYPE_RESULT_OK, 
    				BaseMessages.getString(PKG, "HelloworldMeta.CHECK_OK_FIELD"), 
    				stepMeta
    			);
    			remarks.add(ok);//把验证结果添加到检验转换时出现的问题列表里。
    		}
    	}
    
    	/**
    	 *	creates a new instance of the step (factory)
    	 * getStep、getStepData和getDialogClassName()方法提供了与这个步骤里其它三个接口之间的桥梁
    	 这个接口里还定义了几个方法来说明这四个接口如何结合到一起。
    	String getDialogClassName():用来描述实现了StepDialogInterace接口的对话框类的名字。如果这个方法返回
    				了null，调用类会根据实现了StepMetaInterface接口的类的类名和包名来自动生成对话框类的名字。
    	StepInterface getStep():创建一个实现了StepInterface接口的类。
    	StepInterface getStepData():创建一个实现了StepDataInterface接口的类。
    
    	 */
    	public StepInterface getStep(StepMeta stepMeta, StepDataInterface stepDataInterface,
    			int copyNr, TransMeta transMeta, Trans trans) {
    		return new HelloworldStep(stepMeta, stepDataInterface, copyNr, transMeta, trans);
    	}
    
    	/**
    	 * creates new instance of the step data (factory)
    	 * getStep、getStepData和getDialogClassName()方法提供了与这个步骤里其它三个接口之间的桥梁
    	 */
    	public StepDataInterface getStepData() {
    		return new HelloworldStepData();
    	}
    	/**
    	 * getStep、getStepData和getDialogClassName()方法提供了与这个步骤里其它三个接口之间的桥梁
    	 */
    	@Override
    	public String getDialogClassName() {
    		return HelloworldStepDialog.class.getName();
    	}
    
    	/**
    	 * deserialize from xml 
    	 * databases = list of available connections
    	 * counters = list of sequence steps
    	 * 
    	 * 下面的四个方法loadXML()、getXML()、readRep()和saveRep()把元数据保存到XML文件或资源库里，
    	 * 或者从XML文件或资源库读取元数据。保存到文件的方法利用了像XStream（http://xstream.codehaus.org）这
    	 * 样的XML串行化技术。
    	 */
    	public void loadXML(Node stepDomNode, List<DatabaseMeta> databases,
    			Map<String, Counter> sequenceCounters) throws KettleXMLException {
    		fieldName = XMLHandler.getTagValue(stepDomNode, Tag.field_name.name());
    	}
    	
    	/**
    	 * @Override
    	 */
    	public String getXML() throws KettleException {
    		StringBuilder xml = new StringBuilder();
    		xml.append(XMLHandler.addTagValue(Tag.field_name.name(), fieldName));
    		return xml.toString();
    	}
    	
    	/**
    	 * De-serialize from repository (see loadXML)
    	 */
    	public void readRep(Repository repository, ObjectId stepIdInRepository,
    			List<DatabaseMeta> databases, Map<String, Counter> sequenceCounters)
    			throws KettleException {
    		fieldName = repository.getStepAttributeString(stepIdInRepository, Tag.field_name.name());
    	}
    
    	/**
    	 * serialize to repository
    	 */
    	public void saveRep(Repository repository, ObjectId idOfTransformation, ObjectId idOfStep)
    			throws KettleException {
    		repository.saveStepAttribute(idOfTransformation, idOfStep, Tag.field_name.name(), fieldName);
    	}
    	
    	
    	/**
    	 * initiailize parameters to default
    	 */
    	public void setDefault() {
    		fieldName = "helloField";
    	}
    
    	/**
    	 * getFields()方法非常重要，因为它描述了输出数据行的结构。这个方法需要修改inputRowMeta对象，使这个对象和
    	 * 输出格式匹配。Spoon和后面的步骤都需要知道这个步骤要输出哪些字段。最常见的一种方法，可以给输出的RowMetaInterface对象
    	 * 添加一个ValueMetaInterface对象。在ValueMetaInterface对象里设置的信息越详细越好，可以设置的信息包括数据类型、长度、
    	 * 精度、格式掩码，等等。添加的字段描述元信息越多，后面生成的SQL就越准确。
    	 */
    	@Override
    	public void getFields(RowMetaInterface inputRowMeta, String name,
    			RowMetaInterface[] info, StepMeta nextStep, VariableSpace space)
    			throws KettleStepException {
    		String realFieldName = space.environmentSubstitute(fieldName);
    		//值的元数据使用ValueMetaInterface接口描述数据流里的一个字段。这个接口里定义了字段的名字、数据类型、长度、精度，等等。下面的例子用于创建一个ValueMetaInterface对象。
    		ValueMetaInterface field = new ValueMeta(realFieldName, ValueMetaInterface.TYPE_STRING);
    		field.setOrigin(name);		
    		inputRowMeta.addValueMeta(field);
    	}
    }



代码解析
    @Step(
    		id="Helloworld",
    		name="name",
    		description="description",
    		categoryDescription="categoryDescription", 
    		image="org/kettlesolutions/plugin/step/helloworld/HelloWorld.png",
    		i18nPackageName="org.kettlesolutions.plugin.step.helloworld"
    )
这段代码里的@Step annotation用来通知Kettle的插件注册系统：这个类是一个步骤类型的插件。在annotation里可以指定插件的ID、图标、国际代的包、本地化的名称、类别、描述。其中后三项是资源文件里的Key，需要在资源文件里设置真正的值。i18nPackageName指定了资源文件的包名，例如我们这个例子的资源文件位于org/kettlesolutions/plugin/step/helloworld/messages目录下，en_US（英语，美国）的本地代资源文件是messages_en_US.properties。我们例子里的这个资源文件的内容是：
name=Hello world
description=A very simple step that adds a new "Helllo world" field to the incoming stream
注意，如果你指定了不存在的分类，Spoon会创建这个分类，并在Spoon的分类树的最上方显示这个分类。
最后，annotation里的image标签指定了插件的图标。需要32*32像素的PNG文件，可以使用透明样式。
后面的代码行说明这个类实现了StepMetaInterface接口。在BaseStepMeta抽象类里定义了这个接口的很多默认实现，可以直接继承这个抽象类，然后把工作集中在插件特有的功能上。

    public class HelloworldStepMeta extends BaseStepMeta implements StepMetaInterface
	


下面的四个方法loadXML()、getXML()、readRep()和saveRep()把元数据保存到XML文件或资源库里，或者从XML文件或资源库读取元数据。保存到文件的方法利用了像XStream（http://xstream.codehaus.org）这样的XML串行化技术。


getFields()方法非常重要，因为它描述了输出数据行的结构。这个方法需要修改inputRowMeta对象，使这个对象和输出格式匹配。Spoon和后面的步骤都需要知道这个步骤要输出哪些字段。最常见的一种方法，可以给输出的RowMetaInterface对象添加一个ValueMetaInterface对象。在ValueMetaInterface对象里设置的信息越详细越好，可以设置的信息包括数据类型、长度、精度、格式掩码，等等。添加的字段描述元信息越多，后面生成的SQL就越准确。
####  ####值的元数据（Value Metadata）
值的元数据使用ValueMetaInterface接口描述数据流里的一个字段。这个接口里定义了字段的名字、数据类型、长度、精度，等等。下面的例子用于创建一个ValueMetaInterface对象。  
    ValueMetaInterface dateMeta = new ValueMeta(“birthdate”,ValueMetaInterface.TYPE_DATE);
这个接口也负责转换数据格式。我们建议使用ValueMetaInterface接口来完成所有数据转换的工作。例如，日期类型的数据，如果想把它转换为dateMeta对象里定义的字符串格式，可以用下面的代码：  
    //java.util.Date birthdate
    String birthDateString = dateMeta.getString(birthdate);
ValueMeta类负责转换。因为有ValueMetaInterface进行数据类型的转换，所以你不用再去做额外的数据类型转换的工作。  
使用ValueMetaInterface接口时还要注意数据对象是否为Null。从上一个步骤可以接收到一个数据对象和一个描述数据对象的ValueMetaInterface对象。我们要检查这个数据对象是否为null，在某些情况下如果数据对象为空是不正确的。例如：  
数据对象是String类型，有10个空格，Value Metadata需要trim这个字符串。  
在Value Metadata里已经定义了从文本文件里加载的数据，要延迟转换为字符串。所以数据要由二进制的格式（原始数据格式），转换为字符串格式，然后再转换为其它格式的数据。  
一般使用下面的方法检查数据对象是否为空：  
    Boolean n = valueMeta.isNull(valueDate);
重要：要保证传给ValueMetaInterface对象的数据是在元数据里定义的数据类型。表23-1说明了  ValueMetaInterface里定义的数据类型和Java数据类型的对应关系。  
Kettle元数据类型和Java里数据类型的对应关系  
<table>
    <tr>
        <th>Value Meta Type</th>
        <th>Java Class</th>
    </tr>
    <tr>
        <td>ValueMetaInterface.TYPE_STRING</td>
        <td>Java.lang.String</td>
    </tr>
    <tr>
        <td>ValueMetaInterface.TYPE_DATE</td>
        <td>Java.util.Date</td>
    </tr>
    <tr>
        <td>ValueMetaInterface.TYPE_BOOLEAN</td>
        <td>Java.lang.Boolean</td>
    </tr>
	<tr>
        <td>ValueMetaInterface.TYPE_NUMBER</td>
        <td>Java.lang.Double</td>
    </tr>
	<tr>
        <td>ValueMetaInterface.TYPE_INTEGER</td>
        <td>Java.lang.Long</td>
    </tr>
	<tr>
        <td>ValueMetaInterface.TYPE_BIGNUMBER</td>
        <td>Java.math.BigDecimal</td>
    </tr>
	<tr>
        <td>ValueMetaInterface.TYPE_BINARY</td>
        <td>Byte[]</td>
    </tr>
</table>


####  ####行的元数据（Row Meatadata）
行的元数据使用RowMetaInterface接口来描述数据行的元数据，而不是一个列的元数据。实际上，RowMetaInterface的类里包含了一组ValueMetaInterface。另外还包括了一些方法来操作行元数据，倒如查询值、检查值是否存、替换值的元数据等。  
行的元数据里唯一的规则就是一行里的列的名字必须唯一。当你添加了一个新列时，如果新列的名字和已有列的名字相同，列名后面会自动加上“_2”后缀。如果再加一个同名的列会自动加上”_3“后缀，等等。  
因为在步骤里通常是和数据行打交道，所以从数据行里直接取数据会更方便。可以使用很多类似于getNumber()、getString()这样的方法直接从数据行取数据。例如，销售数据存储在第四列里，可以用下面的代码获取这个数据：  

    Double sales = getInputRowMeta().getNumber(rowData,3);
通过索引获取数据是最快的方式。通过indexOfValue()方法可以获取列在一行里的索引。这个方法扫描列数组，速度并不快。所以，如果要处理所有数据行，我们建议只查询一次列索引。一般是在步骤接收到第一行数据时，就查询列索引，将查询到的列索引保存起来，供后面的数据行使用。  
###  ###StepDatainterface
实现了org.pentaho.di.trans.step.StepDataInterface接口的类用来维护步骤的执行状态，以及存储临时对象。例如，可以把输出行的元数据、数据库连接、输入输出流等存储到这个对象里。  
HelloworldStepData.java  
    package org.kettlesolutions.plugin.step.helloworld;

import org.pentaho.di.core.row.RowMetaInterface;
import org.pentaho.di.trans.step.BaseStepData;
import org.pentaho.di.trans.step.StepDataInterface;

public class HelloworldStepData extends BaseStepData implements StepDataInterface {

	public RowMetaInterface outputRowMeta;

}
    
###  ###StepDialogInterface
实现org.pentaho.di.trans.step.StepDialogInterfac接口的类用来提供一个用户界面，用户通过这个界面输入元数据（转换参数）。用户界面就是一个对话框。这个接口里包含了类似open()和setRepository()等的几个简单的方法。    
####  ####Eclipse SWT
Kettle里使用Eclipse SWT作为界面开发包，所以你也要使用SWT来开发对话框窗口。SWT为不同的操作系统Windows、OS X、Linux和Unix提供了一个抽象层。所以SWT的图形界面和操作系统期货的程序的界面风格非常相近。  
在开始进行SWT开发之前，建议先访问SWT主面以了解更多的内容http://www.eclipse/org/swt。在SWT的网站上，你可以了解到SWT能做出什么样的界面效果：  
SWT控件页，http://www.eclipse.org/swt/widgets/，给出了你能使用的所有控件。  
SWT样例页，http://www.eclipse.org/swt/snippets/，给出了许多代码例子。  
最好的资源就是Kettle里150个内置步骤的对话框源代码。  

HelloworldStepDialog.java  
    package org.kettlesolutions.plugin.step.helloworld;
    
    import org.eclipse.swt.SWT;
    import org.eclipse.swt.events.ModifyEvent;
    import org.eclipse.swt.events.ModifyListener;
    import org.eclipse.swt.events.SelectionAdapter;
    import org.eclipse.swt.events.SelectionEvent;
    import org.eclipse.swt.events.ShellAdapter;
    import org.eclipse.swt.events.ShellEvent;
    import org.eclipse.swt.layout.FormAttachment;
    import org.eclipse.swt.layout.FormData;
    import org.eclipse.swt.layout.FormLayout;
    import org.eclipse.swt.widgets.Button;
    import org.eclipse.swt.widgets.Control;
    import org.eclipse.swt.widgets.Display;
    import org.eclipse.swt.widgets.Event;
    import org.eclipse.swt.widgets.Label;
    import org.eclipse.swt.widgets.Listener;
    import org.eclipse.swt.widgets.Shell;
    import org.eclipse.swt.widgets.Text;
    import org.pentaho.di.core.Const;
    import org.pentaho.di.i18n.BaseMessages;
    import org.pentaho.di.trans.TransMeta;
    import org.pentaho.di.trans.step.BaseStepMeta;
    import org.pentaho.di.trans.step.StepDialogInterface;
    import org.pentaho.di.ui.core.widget.TextVar;
    import org.pentaho.di.ui.trans.step.BaseStepDialog;
    
    public class HelloworldStepDialog extends BaseStepDialog implements
    		StepDialogInterface {
    
    	private static Class<?> PKG = HelloworldStepMeta.class; // for i18n
    															// purposes, needed
    															// by Translator2!!
    															// $NON-NLS-1$
    
    	private HelloworldStepMeta input;
    
    	private TextVar wFieldname;
    
    	public HelloworldStepDialog(Shell parent, Object baseStepMeta,
    			TransMeta transMeta, String stepname) {
    		//初始化元数据对象以及步骤对话框的父类
    		super(parent, (BaseStepMeta) baseStepMeta, transMeta, stepname);
    		input = (HelloworldStepMeta) baseStepMeta;
    	}
    
    	public String open() {
    		Shell parent = getParent();
    		Display display = parent.getDisplay();
    
    		shell = new Shell(parent, SWT.DIALOG_TRIM | SWT.RESIZE | SWT.MIN
    				| SWT.MAX);
    		props.setLook(shell);
    		setShellImage(shell, input);
    
    		ModifyListener lsMod = new ModifyListener() {
    			public void modifyText(ModifyEvent e) {
    				input.setChanged();
    			}
    		};
    		changed = input.hasChanged();
    
    		FormLayout formLayout = new FormLayout();
    		formLayout.marginWidth = Const.FORM_MARGIN;
    		formLayout.marginHeight = Const.FORM_MARGIN;
    
    		shell.setLayout(formLayout);
    		shell.setText(BaseMessages.getString(PKG,
    				"HelloworldDialog.Shell.Title")); //$NON-NLS-1$
    		
    		//所有控件的右侧使用一个自定义的百分对对齐。控件之间的间距使用一个常量，常量值是4像素。
    		int middle = props.getMiddlePct();
    		int margin = Const.MARGIN;
    
    		// Stepname line
    		wlStepname = new Label(shell, SWT.RIGHT);
    		wlStepname.setText(BaseMessages.getString(PKG,
    				"HelloworldDialog.Stepname.Label")); //$NON-NLS-1$
    		props.setLook(wlStepname);
    		fdlStepname = new FormData();
    		fdlStepname.left = new FormAttachment(0, 0);
    		fdlStepname.right = new FormAttachment(middle, -margin);
    		fdlStepname.top = new FormAttachment(0, margin);
    		wlStepname.setLayoutData(fdlStepname);
    		wStepname = new Text(shell, SWT.SINGLE | SWT.LEFT | SWT.BORDER);
    		wStepname.setText(stepname);
    		props.setLook(wStepname);
    		wStepname.addModifyListener(lsMod);
    		fdStepname = new FormData();
    		fdStepname.left = new FormAttachment(middle, 0);
    		fdStepname.top = new FormAttachment(0, margin);
    		fdStepname.right = new FormAttachment(100, 0);
    		wStepname.setLayoutData(fdStepname);
    		Control lastControl = wStepname;
    
    		// Fieldname line
    		//创建一个新的标签控件，控件里文本靠右对齐
    		Label wlFieldname = new Label(shell, SWT.RIGHT);
    		wlFieldname.setText(BaseMessages.getString(PKG,
    				"HelloworldDialog.Fieldname.Label")); //$NON-NLS-1$
    		//下面一行为控件设置用户定义的背景色和字体
    		props.setLook(wlFieldname);
    		FormData fdlFieldname = new FormData();
    		fdlFieldname.left = new FormAttachment(0, 0);
    		fdlFieldname.right = new FormAttachment(middle, -margin);
    		fdlFieldname.top = new FormAttachment(lastControl, margin);
    		wlFieldname.setLayoutData(fdlFieldname);
    		wFieldname = new TextVar(transMeta, shell, SWT.SINGLE | SWT.LEFT
    				| SWT.BORDER);
    		props.setLook(wFieldname);
    		wFieldname.addModifyListener(lsMod);
    		FormData fdFieldname = new FormData();
    		fdFieldname.left = new FormAttachment(middle, 0);
    		fdFieldname.top = new FormAttachment(lastControl, margin);
    		fdFieldname.right = new FormAttachment(100, 0);
    		wFieldname.setLayoutData(fdFieldname);
    		lastControl = wFieldname;
    
    		// Some buttons
    		wOK = new Button(shell, SWT.PUSH);
    		wOK.setText(BaseMessages.getString(PKG, "System.Button.OK")); //$NON-NLS-1$
    		wCancel = new Button(shell, SWT.PUSH);
    		wCancel.setText(BaseMessages.getString(PKG, "System.Button.Cancel")); //$NON-NLS-1$
    
    		setButtonPositions(new Button[] { wOK, wCancel }, margin, lastControl);
    
    		// Add listeners
    		lsCancel = new Listener() {
    			public void handleEvent(Event e) {
    				cancel();
    			}
    		};
    		lsOK = new Listener() {
    			public void handleEvent(Event e) {
    				ok();
    			}
    		};
    
    		wCancel.addListener(SWT.Selection, lsCancel);
    		wOK.addListener(SWT.Selection, lsOK);
    
    		lsDef = new SelectionAdapter() {
    			public void widgetDefaultSelected(SelectionEvent e) {
    				ok();
    			}
    		};
    
    		wStepname.addSelectionListener(lsDef);
    		wFieldname.addSelectionListener(lsDef);
    
    		// Detect X or ALT-F4 or something that kills this window...
    		shell.addShellListener(new ShellAdapter() {//保证了窗口在非正常关闭时，取消用户的编辑
    			public void shellClosed(ShellEvent e) {
    				cancel();
    			}
    		});
    
    		// Populate the data of the controls
    		//下面的代码把数据从步骤的元数据对象里复制到窗口的控件里
    		getData();
    
    		// Set the shell size, based upon previous time...
    		//窗口的大小和位置将根据窗口的自然属性、上次窗口大小和位置，以及显示屏的大小自动设置
    		setSize();
    
    		input.setChanged(changed);
    
    		shell.open();
    		while (!shell.isDisposed()) {
    			if (!display.readAndDispatch())
    				display.sleep();
    		}
    		return stepname;
    	}
    
    	/**
    	 * Copy information from the meta-data input to the dialog fields.
    	 */
    	public void getData() {
    		wStepname.selectAll();
    		//为了防止用户向控件里输入空值，Kettle提供了一个静态方法来检查宿舍，Const.NVL()
    		wFieldname.setText(Const.NVL(input.getFieldName(), ""));
    	}
    
    	private void cancel() {
    		stepname = null;
    		input.setChanged(changed);
    		dispose();
    	}
    	//单击OK把控件里用户输入的数据都写入到步骤的元数据对象中。
    	private void ok() {
    		if (Const.isEmpty(wStepname.getText()))
    			return;
    
    		stepname = wStepname.getText(); // return value
    
    		input.setFieldName(wFieldname.getText());
    
    		dispose();
    	}
    }
    

####  ####窗体布局
如果你看过步骤对话框的源代码，你就会发现窗体类里有很多烦琐的代码。这些代码确保Kettle可以在各种操作系统下以合适的方式展现窗体。可以发现窗体里的大部分代码都和布局以及控件位置有关。  
FormLayout是SWT里经常看到的布局方式。程序员可以通过FormLayout指定控件的百分比、偏移。下面是我们例子里的窗口布局的代码（HelloworldStepDialog.java）  
    //创建一个新的标签控件，控件里文本靠右对齐
    Label label = new Label(shell, SWT.RIGHT);
    label.setText(BaseMessages.getString(PKG,"HelloworldDialog.Fieldname.Label")); //$NON-NLS-1$
    //下面一行为控件设置用户定义的背景色和字体
    props.setLook(label);
    /**
    * 下面几行将标签的左侧和对话框的最左侧对齐，把标签的右侧放在对话框中间（50%）的左侧10个像素
    * 的位置。标签的顶部放在距离对话框顶部25个像素的位置。
    */
    FormData fdLabel = new FormData();
    fdlFieldname.left = new FormAttachment(0, 0);
    fdlFieldname.right = new FormAttachment(50, -10);
    fdlFieldname.top = new FormAttachment(0, 25);
    wlFieldname.setLayoutData(fdLabel);  
简而言之，不要感到痛苦；图形用户界面的代码都比较烦琐，但代码并不复杂。  
####  ####Kettle UI元素  
除了标准的SWT组件，还可以使用Kettle自带的一些控件，Kettle开发人员的工作可以更简单一些。Kettle自带的组件包括以下一些。  
TableView：这是一个数据表格组件，支持排序、选择、键盘快捷键和撤销/重做，以及右键菜单。  
TextVar：这是一个支持变量的文本输入框，这个输入框的右上角有一个$符号。用户可以通过”Ctrl  +Alt+空格”的方式，在弹出的下拉列表中选择变量。其他功能和普通的文本框相同。  
ComboVar：标准的组合下拉列表，支持变量。  
ConditionEditor：过滤行步骤里使用的输入条件控件。  
另外还有很多常用的对话框帮你完成相应的工作，如下所示:  
EnterListDialog:从字符串列表里选择一个或多个字符串。左侧显示字符串列表，右侧是选中的字符串，并提供把字符串从左侧移动到右侧的按钮。  
EnterNumberDialog:用户可以输入数字  
EnterPasswordDialog:让用户输入密码  
EnterSelectionDialog:通过高亮显示，从列表里选择多项  
EnterMappingDialog:输入两组字符串的映射  
PreviewRowsDialog:在对话框里预览一组数据行。  
SQLEditor:一个简单的SQL编辑器，可以输入查询和DDL.  
ErrorDialog:显示异常信息，列出详细的错误栈对话框  
####  ####Hello World例子对话框
现在我们已经基本了解了SWT以及对话框的布局方式，再看看我们的例子，下面的代码是HelloWorldStepDialog.java里的例子。
代码的第一部分是初始化元数据对象以及步骤对话框的父类：
    public class HelloworldStepDialog extends BaseStepDialog implements
    		StepDialogInterface {
    	private static Class<?> PKG = HelloworldStepMeta.class; 
    	private HelloworldStepMeta input;
    	private TextVar wFieldname;
    	public HelloworldStepDialog(Shell parent, Object baseStepMeta,
    			TransMeta transMeta, String stepname) {
    		//初始化元数据对象以及步骤对话框的父类
    		super(parent, (BaseStepMeta) baseStepMeta, transMeta, stepname);
    		input = (HelloworldStepMeta) baseStepMeta;
    	}
在下面的open()方法里创建对话框里的所有控件。SWT使用事件监听模式，可以为控件创建各种监听方法，以响应控件内容的变化和用户的动作。
    public String open() {
    		Shell parent = getParent();
    		Display display = parent.getDisplay();
    		shell = new Shell(parent, SWT.DIALOG_TRIM | SWT.RESIZE | SWT.MIN
    				| SWT.MAX);
    		props.setLook(shell);
    		setShellImage(shell, input);
    		ModifyListener lsMod = new ModifyListener() {
    			public void modifyText(ModifyEvent e) {
    				input.setChanged();
    			}
    		};
    		changed = input.hasChanged();
    
下面代码说明窗体里的控件将使用formLayout的布局方式：
    FormLayout formLayout = new FormLayout();
    		formLayout.marginWidth = Const.FORM_MARGIN;
    		formLayout.marginHeight = Const.FORM_MARGIN;
    		shell.setLayout(formLayout);
所有控件的右侧使用一个自定义的百分比对齐：props.getMiddlePct()；控件之间的间距使用一个常量，常量值是4像素。
    shell.setLayout(formLayout);
    		shell.setText(BaseMessages.getString(PKG,
    				"HelloworldDialog.Shell.Title")); //$NON-NLS-1$
    		int middle = props.getMiddlePct();
    		int margin = Const.MARGIN;
下面的代码在对话框的最上面添加了一行步骤名称标签和输入文本框：
    // Stepname line
    		wlStepname = new Label(shell, SWT.RIGHT);
    		wlStepname.setText(BaseMessages.getString(PKG,
    				"HelloworldDialog.Stepname.Label")); //$NON-NLS-1$
    		props.setLook(wlStepname);
    		fdlStepname = new FormData();
    		fdlStepname.left = new FormAttachment(0, 0);
    		fdlStepname.right = new FormAttachment(middle, -margin);
    		fdlStepname.top = new FormAttachment(0, margin);
    		wlStepname.setLayoutData(fdlStepname);
    		wStepname = new Text(shell, SWT.SINGLE | SWT.LEFT | SWT.BORDER);
    		wStepname.setText(stepname);
    		props.setLook(wStepname);
    		wStepname.addModifyListener(lsMod);
    		fdStepname = new FormData();
    		fdStepname.left = new FormAttachment(middle, 0);
    		fdStepname.top = new FormAttachment(0, margin);
    		fdStepname.right = new FormAttachment(100, 0);
    		wStepname.setLayoutData(fdStepname);
    		Control lastControl = wStepname;

下面是新增输出列的列名设置的输入框：
    // Fieldname line
		//创建一个新的标签控件，控件里文本靠右对齐
		Label wlFieldname = new Label(shell, SWT.RIGHT);
		wlFieldname.setText(BaseMessages.getString(PKG,
				"HelloworldDialog.Fieldname.Label")); //$NON-NLS-1$
		//下面一行为控件设置用户定义的背景色和字体
		props.setLook(wlFieldname);
		FormData fdlFieldname = new FormData();
		fdlFieldname.left = new FormAttachment(0, 0);
		fdlFieldname.right = new FormAttachment(middle, -margin);
		fdlFieldname.top = new FormAttachment(lastControl, margin);
		wlFieldname.setLayoutData(fdlFieldname);
		wFieldname = new TextVar(transMeta, shell, SWT.SINGLE | SWT.LEFT
				| SWT.BORDER);
		props.setLook(wFieldname);
		wFieldname.addModifyListener(lsMod);
		FormData fdFieldname = new FormData();
		fdFieldname.left = new FormAttachment(middle, 0);
		fdFieldname.top = new FormAttachment(lastControl, margin);
		fdFieldname.right = new FormAttachment(100, 0);
		wFieldname.setLayoutData(fdFieldname);
		lastControl = wFieldname;
    
然后创建两个按钮，“确认”和“取消”按钮，以及按钮单击事件的监听方法，把按钮放在对话框的最下面：
    // Some buttons
		wOK = new Button(shell, SWT.PUSH);
		wOK.setText(BaseMessages.getString(PKG, "System.Button.OK")); //$NON-NLS-1$
		wCancel = new Button(shell, SWT.PUSH);
		wCancel.setText(BaseMessages.getString(PKG, "System.Button.Cancel")); //$NON-NLS-1$

		setButtonPositions(new Button[] { wOK, wCancel }, margin, lastControl);

		// Add listeners
		lsCancel = new Listener() {
			public void handleEvent(Event e) {
				cancel();
			}
		};
		lsOK = new Listener() {
			public void handleEvent(Event e) {
				ok();
			}
		};
		wCancel.addListener(SWT.Selection, lsCancel);
		wOK.addListener(SWT.Selection, lsOK);
    
下面的代码做了两件事情，上部代码可以保证当步骤名称或输出字段名称的输入框在编辑状态时，单击“确定”按钮，正在编辑的内容不会丢失；下部的代码保证了窗口在非正常关闭时（没有使用“确定”或“取消”按钮关闭），取消用户的编辑。
    lsDef = new SelectionAdapter() {
			public void widgetDefaultSelected(SelectionEvent e) {
				ok();
			}
		};

		wStepname.addSelectionListener(lsDef);
		wFieldname.addSelectionListener(lsDef);

		// Detect X or ALT-F4 or something that kills this window...
		shell.addShellListener(new ShellAdapter() {//保证了窗口在非正常关闭时，取消用户的编辑
			public void shellClosed(ShellEvent e) {
				cancel();
			}
		});


下面的代码把数据从步骤的元数据对象里复制到窗口的控件里：
    // Populate the data of the controls
    		getData();
窗口的大小和位置将根据窗口的自然属性、上次窗口大小和位置，以及显示屏的大小自动设置。
    // Set the shell size, based upon previous time...
    		//窗口的大小和位置将根据窗口的自然属性、上次窗口大小和位置，以及显示屏的大小自动设置
    		setSize();
    		input.setChanged(changed);
    
    		shell.open();
    		while (!shell.isDisposed()) {
    			if (!display.readAndDispatch())
    				display.sleep();
    		}
    		return stepname;
    	}
为了防止用户身控件里输入空值，Kettle提供了一个静态方法来检查空值，ConstNVL();
    /**
    	 * Copy information from the meta-data input to the dialog fields.
    	 */
    	public void getData() {
    		wStepname.selectAll();
    		wFieldname.setText(Const.NVL(input.getFieldName(), ""));
    	}
最后，单击OK按钮后，把控件里用户输入的数据都写入到步骤的元数据对象中：
    private void cancel() {
    		stepname = null;
    		input.setChanged(changed);
    		dispose();
    	}
    	//单击OK把控件里用户输入的数据都写入到步骤的元数据对象中。
    	private void ok() {
    		if (Const.isEmpty(wStepname.getText()))
    			return;
    
    		stepname = wStepname.getText(); // return value
    
    		input.setFieldName(wFieldname.getText());
    
    		dispose();
    	}

###  ###StepInteface
	这个类实现了org.pentaho.di.trans.step.StepInterface接口，这个类读取上个步骤传来的数据行，利用StepMetaInterface对象里定义的元数据，逐行转换和处理上个步骤传来的数据行，Kettle引擎直接使用这个接口里的很多方法来执行转换过程，但大部分方法都已经由BaseStep类实现了，通常开发人员只需要重载其中的几个方法。
	Init():步骤初始化方法，用来初始化一个步骤。初始化结果是一个true或者false的Boolean值。如果你的步骤没有任何初始化的工作，可以不用重载这个方法。
	Dispose():如果有需要释放的资源，可以在dispose()方法里释放，例如可以关闭数据库连接、释放文件、清除缓存等。在转换的最后Kettle引擎会调用这个方法。如果没有需要释放或清除的资源，可以不用重载这个方法。
	processRow():这个方法，是步骤实现工作的地方。只要这个方法返回true，转换引擎就会重复调用这个方法。
下面是HellWorld例子实现的StepInterface接口（HelloworldStep.java）

HelloworldStep.java
    package org.kettlesolutions.plugin.step.helloworld;
    
    import org.pentaho.di.core.exception.KettleException;
    import org.pentaho.di.core.row.RowDataUtil;
    import org.pentaho.di.trans.Trans;
    import org.pentaho.di.trans.TransMeta;
    import org.pentaho.di.trans.step.BaseStep;
    import org.pentaho.di.trans.step.StepDataInterface;
    import org.pentaho.di.trans.step.StepInterface;
    import org.pentaho.di.trans.step.StepMeta;
    import org.pentaho.di.trans.step.StepMetaInterface;
    /**
     * BaseStep抽象类已经实现了接口里的很多方法，我们只要覆盖需要修改的方法即可。
     * @author Administrator
     *
     */
    public class HelloworldStep extends BaseStep implements StepInterface {
    	/**
    	 * 类的构造函数通常直接把参数传递给BaseStep父类。由父类里的方法来构造对象，然后可以直接
    	 * 使用类似transMeta这样的对象。
    	 * @param stepMeta
    	 * @param stepDataInterface
    	 * @param copyNr
    	 * @param transMeta
    	 * @param trans
    	 */
    	public HelloworldStep(StepMeta stepMeta, StepDataInterface stepDataInterface,
    			int copyNr, TransMeta transMeta, Trans trans) {
    		super(stepMeta, stepDataInterface, copyNr, transMeta, trans);
    		// TODO Auto-generated constructor stub
    	}
    
    	
    	public boolean processRow(StepMetaInterface smi, StepDataInterface sdi) throws KettleException {
    
    		HelloworldStepMeta meta  = (HelloworldStepMeta) smi;
    		HelloworldStepData data = (HelloworldStepData) sdi;
    		/**
    		 * getRow()方法从上一个步骤获取一行数据。如果没有更多要获取的数据行，这个方法就会返回null。
    		 * 如果前面的步骤不能及时提供数据，这个方法就会阻塞，直到有可用的数据行。这样这个步骤的速度就会降低，也会影响
    		 * 其它步骤的速度。
    		 */
    		Object[] row = getRow();
    		if (row==null) {
    			/**
    			 * setOutputDone()方法用来通知其它的步骤，本步骤已经没有输出数据行。下一个步骤如果
    			 * 再调用getRow()方法就会返回null,转换也不再调用processRow()方法。
    			 */
    			setOutputDone();
    			return false;
    		}
    		
    		if (first) {
    			first=false;
    			/**
    			 * 从性能上考虑，getRow()方法不提供数据行的元数据，只提供上个步骤输出的数据。可以使用getInputRowMeta()方法
    			获取元数据，元数据只获取一次即可，所以在first代码块里获取元数据。
    			   如果要把数据传到下一个步骤，要使用putRow()方法。除了输出数据，还要输出RowMetaInterface元数据。
    			   第一行使用clone()方法把输入行的元数据结构复制给输出行。输出行的元数据结构是在输入行的基础上增加一个字段，但
    			   构造输出行的元数据结构只能构造一次，因为所有输出数据行的结构都是一样的，产生了输出行以后，元数据结构就不能再变化。
    			   所以输出行的元数据结构在first代码块里构造。first是一个内部成员，first代码块里的代码只在处理第一行数据时执行。
    			   下面代码的最后一行，给输出数据增加了一个字段。
    			 */
    			data.outputRowMeta = getInputRowMeta().clone();
    			meta.getFields(data.outputRowMeta, getStepname(), null, null, this);
    		}
    		/**
    		 * 下面的代码，把数据写入输出流。从性能角度考虑，数据行实现就是Java数组。为了开发方便，可以使用RowDataUtil类提供
    		 * 的一些静态方法来操作数据。使用RowDatautil静态方法复制数据，还可以提高性能。
    		 */
    		String value = "Hello, world!";
    		
    		Object[] outputRow = RowDataUtil.addValueData(row, getInputRowMeta().size(), value);
    		
    		putRow(data.outputRowMeta, outputRow);
    		
    		return true;
    	}
    }
解析：
public class HelloworldStep extends BaseStep implements StepInterface {
BaseStep抽象类已经实现了接口里的很多方法，我们只要覆盖需要修改的方法即可。
类的构造函数通常直接把参数传递给BaseStep父类。由父类里的方法来构造对象，然后可以直接使用类似transMeta这样的对象。
public HelloworldStep(StepMeta stepMeta, StepDataInterface stepDataInterface,
			int copyNr, TransMeta transMeta, Trans trans) {
		super(stepMeta, stepDataInterface, copyNr, transMeta, trans);
	}
getRow()方法从上一个步骤获取一行数据。如果没有更多要获取的数据行，这个方法就会返回null。如果前面的步骤不能及时提供数据，这个方法就会阻塞，直到有可用的数据行。这样这个步骤的速度就会降低，也会影响其它步骤的速度。
public boolean processRow(StepMetaInterface smi, StepDataInterface sdi) throws KettleException {
		HelloworldStepMeta meta  = (HelloworldStepMeta) smi;
		HelloworldStepData data = (HelloworldStepData) sdi;
		Object[] row = getRow();
		if (row==null) {
			setOutputDone();
			return false;
		}
		
		if (first) {
			first=false;
			data.outputRowMeta = getInputRowMeta().clone();
			meta.getFields(data.outputRowMeta, getStepname(), null, null, this);
		}
		String value = "Hello, world!";
		Object[] outputRow = RowDataUtil.addValueData(row, getInputRowMeta().size(), value);
		
		putRow(data.outputRowMeta, outputRow);
		
		return true;
	}
从性能上考虑，getRow()方法不提供数据行的元数据，只提供上个步骤输出的数据。可以使用getInputRowMeta()方法获取元数据，元数据只获取一次即可，所以在first代码块里获取元数据。
setOutputDone()方法用来通知其它的步骤，本步骤已经没有输出数据行。下一个步骤如果再调用getRow()方法就会返回null,转换也不再调用processRow()方法。

    Object[] row = getRow();
    		if (row==null) {
    			setOutputDone();
    			return false;
    		}
   如果要把数据传到下一个步骤，要使用putRow()方法。除了输出数据，还要输出RowMetaInterface元数据。
    data.outputRowMeta = getInputRowMeta().clone();
    meta.getFields(data.outputRowMeta, getStepname(), null, null, this);
第一行使用clone()方法把输入行的元数据结构复制给输出行。输出行的元数据结构是在输入行的基础上增加一个字段，但构造输出行的元数据结构只能构造一次，因为所有输出数据行的结构都是一样的，产生了输出行以后，元数据结构就不能再变化。所以输出行的元数据结构在first代码块里构造。first是一个内部成员，first代码块里的代码只在处理第一行数据时执行。下面代码的最后一行，给输出数据增加了一个字段。

下面的代码，把数据写入输出流。
		String value = "Hello, world!";
		Object[] outputRow = RowDataUtil.addValueData(row, getInputRowMeta().size(), value);
		putRow(data.outputRowMeta, outputRow);
从性能角度考虑，数据行实现就是Java数组。为了开发方便，可以使用RowDataUtil类提供的一些静态方法来操作数据。使用RowDatautil静态方法复制数据，还可以提高性能。
从指定的步骤读取数据行
如果你想从前面的某个指定的步骤读取数据行，例如”流查询“步骤，可以使用getRowFrom()方法。
	   RowSet rowSet = findInputRowSet(Source Step Name);
	   Object[] rowData = getRowFrom(rowSet);
	       还可以通过rowSet对象获得数据行的元数据：
	   RowMetaInterface rowMeta = rowSet.getRowMeta();
把数据行写入指定的步骤
如果想把数据写入到某个特定的步骤，例如”过滤“步骤，可以使用putRowTo()方法
	  RowSet rowSet = findOutputRowSet(Target Step Name);
	  ....
	  putRowTo(outputRowMeta,rowData,rowSet);
很明显，输入和输出的RowSet对象只需获得一次即可，这样才更有效率。
把数据行写入到错误处理步骤
如果想让你的步骤支持错误处理，而且元数据类返回的supportErrorHandling()方法返回了true，就可以把数据输出
	  到错误处理步骤里。下面是使用putError()方法的例子：
	  Object[] rowData = getRow();
	  ...
	  try{
	  	...
	  	putRow(...);
	  }catch(Exception e){
	  	if(getStepMeta().isDoingErrorHandling()){
	  		putError(getInputRowMeta(),rowData,errorCode);
	  	}else{
	  		throw(e);
	  	}
	  }
	  从例子里可以看到，这段代码把错误的行数、错误字段名、消息、错误编码都传递给错误处理步骤。
	  错误处理的其他工作都自动完成了。
#### ####识别一个步骤拷贝
因为一个步骤可以有多份拷贝同时执行，有时需要识别出正在使用的是哪个步骤拷贝，可以用下面几个方法。
	 getCopy():获得拷贝号。拷贝号可以唯一标识出步骤的一个拷贝，拷贝号的聚会范围是0-N，N=getStepMeta().getCopies()-1
	 getUniqueStepNrAcrossSlaves():获得在集群模式下运行的步骤拷贝号。
	 getUniqueStepCountAcrossSlaves():获得在集群模式下运行的步骤拷贝总数。
	 通过这些方法可以把一个步骤的工作分配给多份拷贝去完成。例如”CSV文件输入“和”固定文件输入“步骤里都有并行读取文件的选项，这样可以把读取文件的工作放在多个拷贝里或集群里来完成。
#### ####结果反馈
在调用getRow()和putRow()方法时，引擎会自动计算两类度量值，读行数和写行数。这两类度量值可以在界面或日志中记录下来，以监控程序运行的状态。下面几个方法用来操作这两类度量值。
	incrementLinesRead():增加从前面步骤读取到的行数。
	incrementLinesWritten():增加定稿到后面步骤中的行数。
	incrementLinesInput():增加从文件、数据库、网络等资源读取到的行数
	incrementLinesOutput:增加写入到文件、数据库、网络等资源的行数。
	incrementLinesUpdate():增加更新的行数。
	incrementLinesSkipped()：增加跳过的数据行的行数。
	incrementLinesRejected():增加拒绝的数据行的行数。
	这些度量值用来说明步骤执行的情况。可以在Spoon的转换度量面板里看到，也可以存到日志数据库表里。
	使用addResultFile()方法，可以把步骤用到的文件保留下来，保存到结果文件列表里。结果文件列表可以被其它转换或作业项使用。例如，下面的”CSV文件输入“的代码：
ResultFile resultFile = new ResultFile(
	ResultFile.FILE_TYPE_GENERAL,
	fileObject,
getTransMeta().getName(),
getStepName()
);
resultFile.setComment(“File was read by a Csv Input step”);
addREsultFile(resultFile);
#### ####变量替换
	如果输入框需要支持变量，可以使用environmentSubstritute()方法获取变量。例如，若想在“Hello World”例子的字段名输入框里使用变量，就要把StepMetaInterface里的getFields()方法修改成下面的语句：
String realFiledName = apace. environmentSubstritute(fieldName)；
因为步骤本身是一个VariableSpace对象，所以也可以使用下面的语句做变量替换：String value = environmentSubstritute(meta.getSringWithVariables());
#### ####Apache VFS
Kettle里所有操作文件的步骤，都使用Apache VFS系统的方式操作。Apache VFS不但可以从文件系统读取文件（如java.io.File），还可以从很多其他来源读取文件，如FTP服务器、Z学压缩文件，等 等 。  
Apache VFS里的FileObject对象提供了文件的抽象层，然后在Kettle的KettleVFS类里还提供了一系列的静态方法，来更方便使用FileObject对象，例如下面的代码 ：  

    FileObject fileObject = KettleVFS.getFileObject(“zip:http://www.example.com/archive.zip!file.txt”);
`String value = environmentSubstritute(meta.getSringWithVariables());`


应该尽可能多地使用KettleVFS,因为它解决了或饶过了很多Apache VFS目前已知的问题。它也增强了SFTP协议。
#### ####步骤插件部署
部署之前，要把四个Java源代码文件编译为class文件。把编译好的class文件放到一个Jar包里。可以使用IDE来做这些事情，也可以手工使用ant脚本来做这些事情。  
.jar文件应该放在Kettle的plugins/steps目录下。也可以使用一个子目录，把所有的依赖的jar包放在插件jar包所在目录的/lib目录下，不必再放Kettle的类路径中（Kettle的libext/目录）已经有了的jar包。另外可以把多个插件放在一个jar包里。
如果想在IDE里调试插件，可以把插件元数据类的名字放在Kettle_PLUGIN_CLASSES变量里（一个逗号分隔的列表）。关于这个主题的更多信息，请参考pentaho Wiki:http://wiki.pentaho.com/display/EAI/How+to+debug+a+Kettle+4+plugin 。

