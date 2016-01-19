---
layout: post
title: 《研磨设计模式》生成器模式-Builder
categories: [设计模式]
description: 设计模式|生成器模式
keywords: 设计模式,生成器,Builder
autotoc: true
comments: true
---

本文是《研磨设计模式》第八章节学习笔记。

本文讲述内容的完整代码实例见 <https://github.com/sunailian/TouchHigh>。

##一、使用场景

导出数据至文件，可以导出到txt,xml等不同格式的文件

   
##二、代码实战

###不使用生成器模式写法

```java
package Builder.one;
/**
 * 
 * 描述输出数据的对象
 * 
 * @description
 * 
 * @classname ExportDataModel
 * @author sunhanbin
 * @date 2015年7月11日 上午11:32:13
 * @version 1.0
 */
public class ExportDataModel {
    private String productId;
    private double price;
    private double amount;
    public String getProductId() {
        return productId;
    }
    public void setProductId(String productId) {
        this.productId = productId;
    }
    public double getPrice() {
        return price;
    }
    public void setPrice(double price) {
        this.price = price;
    }
    public double getAmount() {
        return amount;
    }
    public void setAmount(double amount) {
        this.amount = amount;
    }
}
```

```java

/**
 * 
 */
package Builder.one;
/**
 * 描述输出到文件尾的内容的对象
 * 
 * @description
 * 
 * @classname ExportFooterModel
 * @author sunhanbin
 * @date 2015年7月11日 上午11:33:51
 * @version 1.0
 */
public class ExportFooterModel {
	private String exportUser;
	public String getExportUser() {
		return exportUser;
	}
	public void setExportUser(String exportUser) {
		this.exportUser = exportUser;
	}
}
```

```java
package Builder.one;
/**
 * 描述输出到文件头的内容对象
 * 
 * @description
 * 
 * @classname ExportHeaderModel
 * @author sunhanbin
 * @date 2015年7月11日 上午11:32:33
 * @version 1.0
 */
public class ExportHeaderModel {
    private String depId;
    private String exportDate;
    public String getDepId() {
        return depId;
    }
    public void setDepId(String depId) {
        this.depId = depId;
    }
    public String getExportDate() {
        return exportDate;
    }
    public void setExportDate(String exportDate) {
        this.exportDate = exportDate;
    }
}
```


生成TXT的方法：

```java
package Builder.one;
import java.util.Collection;
import java.util.Map;
/**
 * 导出数据到txt文件
 * 
 * @description
 * 
 * @classname ExportToTxt
 * @author sunhanbin
 * @date 2015年7月11日 上午11:34:59
 * @version 1.0
 */
public class ExportToTxt {
    /**
     * 导出数据到文本文件
     * 
     * @description
     * @title export
     * @author sunhanbin
     * @date 2015年7月11日 上午11:35:21
     * @param
     * @return void
     */
    public void export(ExportHeaderModel ehm,
            Map<String, Collection<ExportDataModel>> mapData,
            ExportFooterModel efm) {
        StringBuffer buffer = new StringBuffer();
        // 1、先拼接文件头内容
        buffer.append(ehm.getDepId() + "," + ehm.getExportDate()+"\n");
        // 2、拼接文件内容
        for (String tableName : mapData.keySet()) {
            // 先拼接表名称
            buffer.append(tableName + "\n");
            // 拼接数据
            for (ExportDataModel edm : mapData.get(tableName)) {
                buffer.append(edm.getProductId() + "," + edm.getPrice() + ","
                        + edm.getAmount() + "\n");
            }
        }
        // 3、拼接文件尾的内容
        buffer.append(efm.getExportUser());
        /** 省略文件读写过程 **/
        System.out.println("输出文本文件的内容：\n" + buffer);
    }
}
```

生成XML的方法

```java
package Builder.one;
import java.util.Collection;
import java.util.Map;
public class ExportToXml {
    /**
     * 数据导出到XML文件
     * 
     * @description
     * @title export
     * @author sunhanbin
     * @date 2015年7月11日 下午12:05:02
     * @param @param ehm
     * @param @param mapData
     * @param @param efm
     * @return void
     */
    public void export(ExportHeaderModel ehm,
            Map<String, Collection<ExportDataModel>> mapData,
            ExportFooterModel efm) {
        StringBuffer buffer = new StringBuffer();
        // 拼接文件头
        buffer.append("<?xml version='1.0' encoding='gb2312'?>\n");
        buffer.append("<Report>\n");
        buffer.append("<Header>\n");
        buffer.append("<DepId>" + ehm.getDepId() + "</DepId>\n");
        buffer.append("<ExportDate>" + ehm.getExportDate() + "</ExportDate>\n");
        buffer.append("</Header>\n");
        // 拼接文件数据
        buffer.append("<Body>\n");
        for (String tableName : mapData.keySet()) {
            buffer.append(" <Datas TableName=\"" + tableName + "\">\n");
            for (ExportDataModel edm : mapData.get(tableName)) {
                buffer.append(" <Data>\n");
                buffer.append("   <ProductId>" + edm.getProductId()
                        + "</ProductId>\n");
                buffer.append("   <Price>" + edm.getPrice() + "</Price>\n");
                buffer.append("   <Amount>" + edm.getAmount() + "</Amount>");
                buffer.append("</Data>\n");
            }
            buffer.append(" </Datas>\n");
        }
        buffer.append("</Body>\n");
        // 3、拼接文件尾的内容
        buffer.append("<Footer>\n");
        buffer.append("  <ExportUser>" + efm.getExportUser() + "</ExportUser>\n");
        buffer.append("</Footer>\n");
        buffer.append("</Report>\n");
        /** 省略文件读写过程 **/
        System.out.println("输出文本文件的内容：\n" + buffer);
    }
}
```


##三、生成器模式定义

- 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

- 设计思路：将构建过程独立出来，即模式中的【指导者】。由指导者来指导装配过程，但不负责每步的具体实现
将能实现每步的对象称为【生成器】

![](/images/posts/designpattern/Builder-1.png)

###使用生成器模式改写代码

**Builder:**

```java
/**
 * 
 */
package Builder.two;
import java.util.Collection;
import java.util.Map;
import Builder.one.ExportDataModel;
import Builder.one.ExportFooterModel;
import Builder.one.ExportHeaderModel;
/**
 * 生成器接口，定义创建一个输出文件对象所需的各个部件的操作
 * 
 * @description
 * 
 * @classname Builder
 * @author sunhanbin
 * @date 2015年7月11日 下午12:50:37
 * @version 1.0
 */
public interface Builder {
	
	
	/**
	 * 构建输出文件的Header部分
	 *  
	 * @description
	 * @title  buildHeader
	 * @author  sunhanbin
	 * @date         2015年7月11日 下午12:51:56 
	 * @param  @param edm
	 * @return void
	 */
	public void buildHeader(ExportHeaderModel ehm);
	
	
	/**
	 * 构建输出文件的Body部分
	 *  
	 * @description
	 * @title  buildBody
	 * @author  sunhanbin
	 * @date         2015年7月11日 下午12:52:28 
	 * @param  @param mapData
	 * @return void
	 */
	public void buildBody(Map<String, Collection<ExportDataModel>> mapData);
	
	
	/**
	 * 构建输出文件的Footer部分
	 *  
	 * @description
	 * @title  buildFooter
	 * @author  sunhanbin
	 * @date         2015年7月11日 下午12:52:57 
	 * @param  @param efm
	 * @return void
	 */
	public void buildFooter(ExportFooterModel efm);
}
```

**ConcreteBuilder：Builder的具体实现**

```java
package Builder.two;
import java.util.Collection;
import java.util.Map;
import Builder.one.ExportDataModel;
import Builder.one.ExportFooterModel;
import Builder.one.ExportHeaderModel;
/**
 * 实现导出数据到文本文件的生成器对象
 * 
 * @description
 * 
 * @classname TextBuilder
 * @author sunhanbin
 * @date 2015年7月11日 下午1:06:22
 * @version 1.0
 */
public class TxtBuilder implements Builder {
	private StringBuffer buffer = new StringBuffer();
	public void buildHeader(ExportHeaderModel ehm) {
		buffer.append(ehm.getDepId() + "," + ehm.getExportDate() + "\n");
	}
	public void buildBody(Map<String, Collection<ExportDataModel>> mapData) {
		for (String tableName : mapData.keySet()) {
			// 先拼接表名称
			buffer.append(tableName + "\n");
			// 拼接数据
			for (ExportDataModel edm : mapData.get(tableName)) {
				buffer.append(edm.getProductId() + "," + edm.getPrice() + ","
						+ edm.getAmount() + "\n");
			}
		}
	}
	public void buildFooter(ExportFooterModel efm) {
		buffer.append(efm.getExportUser());
	}
	
	public StringBuffer getResult(){
		return buffer;
	}
}
```

```java
package Builder.two;
import java.util.Collection;
import java.util.Map;
import Builder.one.ExportDataModel;
import Builder.one.ExportFooterModel;
import Builder.one.ExportHeaderModel;
/**
 * 实现导出数据到XML文件的生成器对象
 * 
 * @description
 * 
 * @classname XmlBuilder
 * @author sunhanbin
 * @date 2015年7月11日 下午1:09:05
 * @version 1.0
 */
public class XmlBuilder implements Builder {
	private StringBuffer buffer = new StringBuffer();
	public void buildHeader(ExportHeaderModel ehm) {
		// 拼接文件头
		buffer.append("<?xml version='1.0' encoding='gb2312'?>\n");
		buffer.append("<Report>\n");
		buffer.append("<Header>\n");
		buffer.append("<DepId>" + ehm.getDepId() + "</DepId>\n");
		buffer.append("<ExportDate>" + ehm.getExportDate() + "</ExportDate>\n");
		buffer.append("</Header>\n");
	}
	public void buildBody(Map<String, Collection<ExportDataModel>> mapData) {
		buffer.append("<Body>\n");
		for (String tableName : mapData.keySet()) {
			buffer.append(" <Datas TableName=\"" + tableName + "\">\n");
			for (ExportDataModel edm : mapData.get(tableName)) {
				buffer.append(" <Data>\n");
				buffer.append("   <ProductId>" + edm.getProductId()
						+ "</ProductId>\n");
				buffer.append("   <Price>" + edm.getPrice() + "</Price>\n");
				buffer.append("   <Amount>" + edm.getAmount() + "</Amount>");
				buffer.append("</Data>\n");
			}
			buffer.append(" </Datas>\n");
		}
		buffer.append("</Body>\n");
	}
	public void buildFooter(ExportFooterModel efm) {
		buffer.append("<Footer>\n");
		buffer.append("  <ExportUser>" + efm.getExportUser()
				+ "</ExportUser>\n");
		buffer.append("</Footer>\n");
		buffer.append("</Report>\n");
	}
	public StringBuffer getResult() {
		return buffer;
	}
}
```

**Director：指导者。用来使用Builder接口，以一个统一的过程来构建所需要的对象**

```java
package Builder.two;
import java.util.Collection;
import java.util.Map;
import Builder.one.ExportDataModel;
import Builder.one.ExportFooterModel;
import Builder.one.ExportHeaderModel;
/**
 * 指导者，指导使用生成器的接口来构建输出的文件的对象
 * 
 * @description
 * 
 * @classname Director
 * @author sunhanbin
 * @date 2015年7月11日 下午1:13:37
 * @version 1.0
 */
public class Director {
    private Builder builder;
    public Director(Builder builder) {
        this.builder = builder;
    }
    /**
     * 指导生成器构建最终的输出的文件的对象
     * 
     * @description
     * @title construct
     * @author sunhanbin
     * @date 2015年7月11日 下午1:37:38
     * @param @param ehm
     * @param @param mapData
     * @param @param efm
     * @return void
     */
    public void construct(ExportHeaderModel ehm,
            Map<String, Collection<ExportDataModel>> mapData,
            ExportFooterModel efm) {
        builder.buildHeader(ehm);
        builder.buildBody(mapData);
        builder.buildFooter(efm);
    }
}
```

**最终调用：**

```java
// 输出到文本
TxtBuilder txtbuilder = new TxtBuilder();
Director director1 = new Director(txtbuilder);
director1.construct(ehm, mapData, efm);
// 输出到XML
XmlBuilder xmlBuilder = new XmlBuilder();
Director director2 = new Director(xmlBuilder);
irector2.construct(ehm, mapData, efm);  
```


##四、生成器模式深入

生成器模式分为重要的两部分：<br/>
- Builder接口：定义如何构建各个部件；
- Director类：指导如何使用Builder接口构建产品；
*把构建器对象和被构建的对象合并：通过将类内联化（Inline Class)*

```java
package Builder.three;
/**
 * 将构建器对象和被构建对象合并
 * 
 * @description
 * 
 * @classname InsuranceContract
 * @author sunhanbin
 * @date 2015年7月11日 下午1:55:34
 * @version 1.0
 */
public class InsuranceContract {
    private String contractId;
    private String personName;
    private String companyName;
    private long beginDate;
    private long endDate;
    private String otherData;
    private InsuranceContract(ConcreateBuilder builder) {
        this.contractId = builder.contractId;
        this.personName = builder.personName;
        this.companyName = builder.companyName;
        this.beginDate = builder.beginDate;
        this.endDate = builder.endDate;
        this.otherData = builder.otherData;
    }
    public void someOperation() {
        System.out.println("Now in Insurance Contract someOperation=="
                + this.contractId);
    }
    public static class ConcreateBuilder {
        private String contractId;
        private String personName;
        private String companyName;
        private long beginDate;
        private long endDate;
        private String otherData;
        public ConcreateBuilder(String contractId, long beginDate, long endDate) {
            this.contractId = contractId;
            this.beginDate = beginDate;
            this.endDate = endDate;
        }
        public ConcreateBuilder setPersonName(String personName) {
            this.personName = personName;
            return this;
        }
        public ConcreateBuilder setCompanyName(String companyName) {
            this.companyName = companyName;
            return this;
        }
        public ConcreateBuilder setOtherData(String otherData) {
            this.otherData = otherData;
            return this;
        }
        /**
         * 构建真正的对象并返回
         * 
         * @description
         * @title build
         * @author sunhanbin
         * @date 2015年7月11日 下午2:03:35
         * @param @return
         * @return InsuranceContract
         */
        public InsuranceContract build() {
            if (contractId == null || contractId.trim().length() == 0) {
                throw new IllegalArgumentException("合同编号不能为空");
            }
            boolean signPerson = personName != null && personName.length() > 0;
            boolean signCompany = companyName != null
                    && companyName.length() > 0;
            if (signPerson && signCompany) {
                throw new IllegalArgumentException("一份保险合同不能同时与人和公司签订");
            }
            if (signPerson == false && signCompany == false) {
                throw new IllegalArgumentException("一份保险合同不能没有签订对象");
            }
            if (beginDate <= 0) {
                throw new IllegalArgumentException("合同必须有保险开始生效的日期");
            }
            if (endDate <= 0) {
                throw new IllegalArgumentException("合同必须有保险失效的日期");
            }
            if (endDate <= beginDate) {
                throw new IllegalArgumentException("保险失效的日期必须大于保险生效的日期");
            }
            return new InsuranceContract(this);
        }
    }
}
```

客户端调用也发生了区别：

```java
package Builder.three;
public class Client {
    public static void main(String[] args) {
        // 创建构建器
        InsuranceContract.ConcreateBuilder builder = new InsuranceContract.ConcreateBuilder(
                "001", 12345l, 67890l);
        // 设置需要的数据，然后构建保险合同对象
        InsuranceContract contract = builder.setPersonName("孙汉斌")
                .setOtherData("其他条例").build();
        // 操作保险合同对象的方法
        contract.someOperation();
    }
}
```