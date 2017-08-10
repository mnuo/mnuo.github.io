---
title: WebService
date: 2017-08-12 17:12:08 
tags: Web Service
category: Web Service 
---
WebService 也叫做XML WEB SERVICE, WebService 是一种可以接收从Internet或者Internet上的其他系统中传递过来的请求, 轻量级的独立的通讯技术。
是通过SOAP在web上提供的软件服务,使用WSDL文件进行说明,并通过UDDI进行注册。

### 1 基本概念
- XML(Extensible Markup Language):扩展型可标记语言。面向短期的临时数据处理,面向万维网络,是SOAP的基础
- SOAP(Simple Object Access Protocol): 简单对象存取协议。是XML WEB SERVICE 的通信协议。当用户通过uddi找到你的WSDL描述文档之后,他通过SOAP调用你建立的web服务中的一个或多个操作。SOAP是XML文档形式的调用方法的规范，它可以支持不同的底层接口，像HTTP(S)或者SMTP
- WSDL：(Web Services Description Language) WSDL 文件是一个 XML 文档，用于说明一组 SOAP 消息以及如何交换这些消息。大多数情况下由软件自动生成和使用。
- UDDI (Universal Description, Discovery, and Integration) 是一个主要针对Web服务供应商和使用者的新项目。在用户能够调用Web服务之前，必须确定这个服务内包含哪些商务方法，找到被调用的接口定义，还要在服务端来编制软件，UDDI是一种根据描述文档来引导系统查找相应服务的机制。UDDI利用SOAP消息机制（标准的XML/HTTP）来发布，编辑，浏览以及查找注册信息。它采用XML格式来封装各种不同类型的数据，并且发送到注册中心或者由注册中心来返回需要的数据。

### 2 根据WSDL生成java代码
- 1,下载apache-cxf-2.6.2在环境变量中配置CXF_HOME 值为E:\gavin\cxf\apache-cxf-3.0.0,在PATH中加入%CXF_HOME%\bin
- 2,输入cmd 进入控制窗口，输入wsdl2java看是否配置成功
- 3,wsdl2java -p [包名] -d [路径] -client[server,all] wsdl地址

### 3 调用
- http方法: 
		String soapRequestOfTargetService=
			"<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:ebiz=\"http://www.cmcc.com.cn/EBIZ/service/EBIZService1.0\">"+
			  " <soapenv:Header/>"+
			  " <soapenv:Body>"+
			   "   <ebiz:resultInfoConfirm>"+
			     "    <ebiz:in0>{1}</ebiz:in0>"+
			    "  </ebiz:resultInfoConfirm>"+
			   "</soapenv:Body>"+
			"</soapenv:Envelope>";
		try {						
			String url = ResourceUtil.getProperty(Constant.SERVICE_Synchronize_Result_Info_SRV);			
			PostMethod postmethod = new PostMethod(url);
			logger.info("加密前的请求xml如下:"+reqXML);
			//报文Base64加密
			reqXML=encryptBASE64(reqXML);
			soapRequestOfTargetService=soapRequestOfTargetService.replace("{1}", reqXML);
			logger.info("加密后请求xml如下:"+soapRequestOfTargetService);
			byte[] b = soapRequestOfTargetService.getBytes("UTF-8"); 
			InputStream is = new ByteArrayInputStream(b, 0, b.length);
			RequestEntity re = new InputStreamRequestEntity(is, b.length, "application/xop+xml; charset=UTF-8; type=\"text/xml\"");
			postmethod.setRequestEntity(re);
			HttpClient httpClient = new HttpClient();
			int statusCode = httpClient.executeMethod(postmethod);
			logger.info("http响应状态码：" + statusCode);
			String soapResponseData = postmethod.getResponseBodyAsString();					
			//logger.info("响应xml如下："+soapResponseData);
			//替换符号
			soapResponseData=soapResponseData.replace("&lt;","<");
			soapResponseData=soapResponseData.replace("&gt;",">");
			logger.info("响应xml如下："+soapResponseData);
			//添加接口返回的信息到数据库中
			logger.info("开始插入结果数据");
			insertResultInfo(soapResponseData, report, jtmdmService);			
		} catch (Exception ex) {
			ex.printStackTrace();
		}
- HttpSOAP
		request = new request();
		List<SCMGroupImportedContractInfoImportSrvInputItem> inputItems = request.getCONTRACTINFOLIST();
		inputItems.add(item);
		//调用接口
		Response srvResponse = port.process(request);
		String errorFlag = srvResponse.getERRORFLAG();
		String errorMessage = srvResponse.getERRORMESSAGE();

