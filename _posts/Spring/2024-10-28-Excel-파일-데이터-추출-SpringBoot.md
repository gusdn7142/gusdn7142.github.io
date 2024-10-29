---
layout: single                    
title:  "[SpringBoot] Excel 파일에서 데이터 추출"    
categories: Spring          
tag: [Backend, External Library]
related_label: " "   #footer
excerpt: "Apache POI, OOXML, HSSF, XSSF" 
---
<hr>

### ⬛︎ 목차  
1. [CSV 파일과 Excel 파일 비교](#CSV)  
2. [Excel 관련 라이브러리 Apache POI 소개](#excel-관련-라이브러리-소개)  
3. [Apache POI 라이브러리를 통한 Excel 파일 데이터 추출 실습 With Java∘Spring Boot](#Apache-POI-라이브러리를-통한-Excel-데이터-추출)  
4. [fix1) Cell이 비어있을때 NullPointerException 발생](#cell이-비어있을때-nullpointerexception-발생)  
5. [fix2) Row가 비어있을때 NullPointerException 발생](#row가-비어있을때-nullpointerexception-발생)  
6. [refactor1) extractDataFrom() 메서드의 ExcelDataInfo 객체 종속성 개선](#extractdatafrom메서드의-exceldatainfo-객체-종속성-개선)  

<br>

Excel과 자주 비교되는게 보통 CSV 파일이다. 
그렇기 때문에 **CSV 파일과 Excel 파일의 차이점**에 대해 먼저 알아볼 것이다.

<br>


**CSV** 

- 특징 : 쉼표를 통해 값을 구분한다.
- 단점 : 서식 지정이 불가능하다.
- 장점 : 파일 크기가 작아 빠른 전송이 가능하며, 텍스트 파일이기 때문에 통신 과정에서 데이터가 손실될 가능성이 적다

**Excel**

- 특징 ; MS에서 개발한 스프레드시트 응용 프로그램으로 .xls 또는 .xlsx 확장자로 구분한다.
- 단점 : 파일 크기가 크기 떄문에 전송 속도가 느릴 수 있고, 바이너리 파일이기 때문에 통신 과정에서 데이터가 손실될 가능성이 있다.
- 장점 : 표, 테두리, 배경색, 굵기, 차트, 시각화 등의 서식 지정이 가능하다.

<br>

고객의 입장에서 테이블 형식의 데이터를 서버에 업로드한다고 가정해보자.

(가장 접근성이 쉽고, 배경색, 테두리 등의 서식 지정으로) 데이터 입력이 편한 Excel 파일을 입력으로 받아 지금부터 서버에서 데이터를 추출해 볼 것이다.

<!-- <br> -->

## 🔰 Excel 관련 라이브러리 소개 {#excel-관련-라이브러리-소개}



### **Apache POI™ - Component Overview**

> *Apache POI is also the master project for developing pure Java ports of file formats based on Office Open XML (ooxml)*  
> ***: Apache POI는 Office Open XML(ooxml)을 기반**으로 하는 파일 형식의 순수 **Java** 포트를 개발하기 위한 마스터 **프로젝트**이다.*
> 

### *What is OOXML?*

> *Office Open XML, also known as OpenXML or OOXML, is an XML-based format for office documents, including word processing documents, spreadsheets, presentations, as well as charts, diagrams, shapes, and other graphical material.*  
> ***: Office Open XML은** OpenXML 또는 OOXML이라고도 하며, 워드 프로세싱 문서, **스프레드시트**, 프레젠테이션, 차트, 다이어그램, 도형 및 기타 그래픽 자료를 포함한 사무용 **문서를 위한 XML 기반 형식**이다.*
> 

### **OOXML-based HSSF and XSSF for Excel documents**

> *HSSF is our port of the Microsoft Excel 97 (-2003) file format (BIFF8) to pure Java. XSSF is our port of the Microsoft Excel XML (2007+) file format (OOXML) to pure Java. SS is a package that provides common support for both formats with a common API*  
> ***: HSSF**는 Microsoft Excel 97(-2003) 파일 형식(.**xls**)을 **Java**에서 **지원**하기 위해 구현된 **포맷**이고, **XSSF**는 Microsoft Excel XML(2007+) 파일 형식(.**xlsx**)을 **Java**에서 **지원**하기 위해 구현된 **포맷**이다. **SS**는 두 형식에 대한 공통 지원을 제공하는 패키지인데, 우리의 실습에서는 이 패키지(”org.apache.poi.ss.usermodel”)를 **사용**할 것이다.*
> 

<br>

## 👨‍💻 Apache POI 라이브러리를 통한 Excel 데이터 추출  {#Apache-POI-라이브러리를-통한-Excel-데이터-추출}


Apache POI Excel 라이브러리의 기본적인 사용법에 대한 자세한 내용은 [이 페이지](https://poi.apache.org/components/spreadsheet/quick-guide.html)를 참고하면 된다. 이번 실습에서는 Excel 파일을 요청받고 Excel 파일의 데이터 추출을 진행할 것이고, 최신 Excel 파일 형식인 .xlsx와 XSSFWorkbook을 사용할 것이다. 

개발 환경은 Intellij, Java 17, SpringBoot 3.x로 진행한다.

### 1️⃣ Dependency 설정

**pom.xml**

```xml
<-- Excel 97-2003 support -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.3.0</version>
</dependency>

<-- Excel 2007+ support -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```
[*Reference1) **Check latest poi version***](https://poi.apache.org/download.html)  
[*Reference2) **What is the difference between poi and poi-ooxml***](https://stackoverflow.com/questions/60217698/what-is-the-difference-between-poi-and-poi-ooxml)  
[*Reference3) **Mvnrepository***](https://mvnrepository.com/search?q=Apache+POI)  

<br>


### 2️⃣ Excel 파일 Request & Excel 데이터 추출

1. **Test Excel File 준비** (file extention : .xlsx)
    
    
    | 이름 | 주소 | 전화번호 | 이메일 | 설명 | 생일 | 직업 |
    | --- | --- | --- | --- | --- | --- | --- |
    | testName | testAddress | 010-xxxx-xxxx | testEamil@test.com | testContent | testBirthDay | testJob |

1. **Excel 파일 데이터와 매핑할 Object 생성**
    
    ```java
    public class ExcelDataInfo {
    
        private String name;
        private String address;
        private String phone;
        private String email;
        private String comment;
        private String birthDay;
        private String job;
    
        //getter(), setter(), toString(), Constructor 생략
    }
    ```
    
2. **Excel 파일을 업로드할 Controller 구현**
    
    ```java
    import com.example.boot3Test.model.ExcelDataInfo;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.*;
    import org.springframework.web.multipart.MultipartFile;
    import java.io.IOException;
    import java.util.List;
    
    @Slf4j
    @RestController
    @RequestMapping("/excelfile")
    public class ExcelController {
    
        private ExcelService excelService;
    
        @Autowired
        public ExcelController(ExcelService excelService) {
            this.excelService = excelService;
        }
    
        @PostMapping("/extractdata")
        public void extractDataFromExcel(@RequestParam("excelFile") MultipartFile excelFile) throws IOException {
            List<ExcelDataInfo> excelDataInfosList = excelService.extractDataFrom(excelFile);
            excelDataInfoList.stream().forEach(excelDataInfo -> {log.info("excelDataInfo = {}",excelDataInfo);});
        }
    }
    ```
    
    > Multipart Request : 메타 정보(파일 이름, 파일 크기, 파일의 MIME 유형 등)와 데이터가 포함된 요청 (ex, [MultipartFile](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/multipart/MultipartFile.html))
    > 
    
3. **Excel 파일에서 데이터를 추출하는 Service 구현**
    
    ```java
    @Slf4j
    @Service
    public class ExcelService {
    
        public List<ExcelDataInfo> extractDataFrom(MultipartFile multipartFile) throws IOException {
    
            //1) CREATE Workbook (==XSSFWorkbook)
            Workbook workbook = WorkbookFactory.create(multipartFile.getInputStream());
    
            //2) GET Sheet
            Sheet worksheet = workbook.getSheetAt(0);
    
            //3) EXTRACT Data
            List<ExcelDataInfo> excelDataInfoList = new ArrayList<>();
    
            //3-1) Extract cellValue by row in excel
            for (int i = 1; i < worksheet.getPhysicalNumberOfRows(); i++) { 
    
                Row row = worksheet.getRow(i);
                ExcelDataInfo excelData = new ExcelDataInfo();
                excelData.updateName(row.getCell(0).getStringCellValue());
                excelData.updateAddress(row.getCell(1).getStringCellValue());
                excelData.updatePhone(row.getCell(2).getStringCellValue());
                excelData.updateEmail(row.getCell(3).getStringCellValue());
                excelData.updateComment(row.getCell(4).getStringCellValue());
                excelData.updateBirthDay(row.getCell(5).getStringCellValue());
                excelData.updateJob(row.getCell(6).getStringCellValue());
    
                excelDataInfoList.add(excelData);
            }
    
            return excelDataInfoList;
        }
    }
    
    ```

    <img src="../../assets/images/Spring/2024-10-28-Excel-파일-데이터-추출/excel_file_format.png" alt="excel_file_format" width="500" height="500">
    <br>
    <br>

1. API 실행 결과 확인
    
    ```
    ExcelDataInfo{name='testName', address='testAddress', phone='010-xxxx-xxxx', email='testEamil@test.com', comment='testContent', birthDay='testBirthDay', job='testJob'}
    ```
    
    위와 같이 이름, 주소, 전화번호, … , 직업 등의 필드가 ExcelDataInfo 객체에 정상적으로 저장된 것을 확인할 수 있다.
    
<br>

### fix1) Cell이 비어있을때 NullPointerException 발생  {#cell이-비어있을때-nullpointerexception-발생}

---
>*java.lang.NullPointerException: Cannot invoke org.apache.poi.ss.usermodel.Cell.getStringCellValue() because the return value of org.apache.poi.ss.usermodel.Row.getCell(int) is null*  
>*: Excel 파일에 아래와 같은 구조로 데이터가 저장되어 있다면 **비어있는 Cell로 인해 데이터 추출시 NullPointerException이 발생**한다*



> 
> 
> 
> | 이름 | 주소 | 전화번호 | 이메일 | 설명 | 생일 | 직업 |
> | --- | --- | --- | --- | --- | --- | --- |
> | testName | testAddress | 010-xxxx-xxxx | testEamil@test.com | testContent | testBirthDay | testJob |
> | testName2 | testAddress2 |  |  |  |  |  |


이 Exception을 해결하기 위해서는 아래와 같이 각 row의 cell에 대한 Null 방지 처리를 해주면 된다.

```java
Row row = worksheet.getRow(i);
ExcelDataInfo excelData = new ExcelDataInfo();
excelData.updateName(row.getCell(0) != null ? row.getCell(0).getStringCellValue() : "");
excelData.updateAddress(row.getCell(1) != null ? row.getCell(1).getStringCellValue() : "");
excelData.updatePhone(row.getCell(2) != null ? row.getCell(2).getStringCellValue() : "");
excelData.updateEmail(row.getCell(3) != null ? row.getCell(3).getStringCellValue() : "");
excelData.updateComment(row.getCell(4) != null ? row.getCell(4).getStringCellValue() : "");
excelData.updateBirthDay(row.getCell(5) != null ? row.getCell(5).getStringCellValue() : "");
excelData.updateJob(row.getCell(6) != null ? row.getCell(6).getStringCellValue() : "");
```

적용 후, API 실행 결과를 확인해 보면 Null이 었던 필드들이 ‘ ’로 치환되어 정상적으로 출력되는 것을 확인할 수 있다.

```
ExcelDataInfo{name='testName', address='testAddress', phone='010-xxxx-xxxx', email='testEamil@test.com', comment='testContent', birthDay='testBirthDay', job='testJob'}
ExcelDataInfo{name='testName2', address='testAddress2', phone='', email='', comment='', birthDay='', job=''}
```

<br>
<br>

### fix2) Row가 비어있을때 NullPointerException 발생  {#row가-비어있을때-nullpointerexception-발생}

---

>*java.lang.NullPointerException: Cannot invoke org.apache.poi.ss.usermodel.Row.getCell(int) because row is null*   
>*: Excel 파일에 아래와 같은 구조로 데이터가 저장되어 있다면 **비어있는 Row로 인해 데이터 추출시 NullPointerException이 발생**한다*



> | 이름 | 주소 | 전화번호 | 이메일 | 설명 | 생일 | 직업 |
> | --- | --- | --- | --- | --- | --- | --- |
> | testName | testAddress | 010-xxxx-xxxx | testEamil@test.com | testContent | testBirthDay | testJob |
> |  |  |  |  |  |  |  |
> |  |  |  |  |  |  |  |
> | testName2 | testAddress2 | 010-pppp-pppp | testEamil2@test2.com | testContent2 | testBirthDay2 | testJob2 |

먼저 worksheet.getPhysicalNumberOfRows() 메서드 출력 결과를 확인해보니 3이 나왔다.

실제로 데이터가 존재하는 0번, 1번, 4번 Row에 대해서만 개수를 count하기 때문에 3이 출력된 것을 알 수 있다.   (⚠주의사항 : worksheet의 row는 0부터 시작된다)

이렇게 되면 비어있는 Row에 대한 예외처리를 해주더라도 데이터가 존재하는 마지막 Row까지 반복 할수 없어 다른 방법을 찾아봐야 했다.

역시나 XSSFSheet 구현체에서 Excel 파일의 마지막 Row 번호를 반환하는 getLastRowNum()를 제공해 주고 있어 이 메서드를 적용해 볼것이다. 

```java
public class XSSFSheet extends POIXMLDocumentPart implements Sheet, OoxmlSheetExtensions {
	public int getLastRowNum() {
            return this._rows.isEmpty() ? -1 : (Integer)this._rows.lastKey();
	}
}
```

[ XSSFSheet의 내장 메서드 getLastRowNum() ]

<br>

아래와 같이 worksheet.getLastRowNum() 만큼 반복하며 Row가 null일때 continue 해줌으로써 Null인 Row를 excelDataInfoList에서 제외시킬 수 있다.  

```java
//3-1) Extract cellValue by row in excel
for (int i = 1; i <= worksheet.getLastRowNum(); i++) {
    Row row = worksheet.getRow(i);
    if(row == null){		
        continue;
    }

    ExcelDataInfo excelData = new ExcelDataInfo();
    excelData.updateName(row.getCell(0).getStringCellValue());
    //생략
    excelData.updateJob(row.getCell(6).getStringCellValue());
    excelDataInfoList.add(excelData);
}
```

아래는 API 실행 결과이다.

```
ExcelDataInfo{name='testName', address='testAddress', phone='010-xxxx-xxxx', email='testEamil@test.com', comment='testContent', birthDay='testBirthDay', job='testJob'}
ExcelDataInfo{name='testName2', address='testAddress2', phone='010-pppp-pppp', email='testEamil2@test2.com', comment='testContent2', birthDay='testBirthDay2', job='testJob2'}
```

<br>
<br>

### refactor1)  extractDataFrom() 메서드의 ExcelDataInfo 객체 종속성 개선 {#extractdatafrom메서드의-exceldatainfo-객체-종속성-개선}

---

**1단계)** 개발자가 직접 ExcelDataInfo 객체의 Field마다 값을 넣어주었던 로직을 제거하고 Reflection을 적용해 Field에 값을 주입

```java
public List<ExcelDataInfo> extractDataFrom(MultipartFile multipartFile) throws IOException, IllegalAccessException {

	//1) CREATE Workbook (==XSSFWorkbook)
	Workbook workbook = WorkbookFactory.create(multipartFile.getInputStream());
	
	//2) GET Sheet
	Sheet worksheet = workbook.getSheetAt(0);
	
	//3) EXTRACT Data
	List<ExcelDataInfo> excelDataInfoList = new ArrayList<>();
	
	//3-1) Extract cellValue by row in excel
	for (int rowIndex = 1; rowIndex <= worksheet.getLastRowNum(); rowIndex++) {
	
	    Row row = worksheet.getRow(rowIndex);
	    if (row == null) {
	        continue;
	    }
		
	    ExcelDataInfo excelData = ExcelDataInfo.builder().build();
	    Field[] fields = excelData.getClass().getDeclaredFields();
		
		for (int cellIndex = 0; cellIndex < fields.length; cellIndex++) {
		    //3-1-1) Get ExcelDataInfo's Field
		    Field field = fields[cellIndex];
		    field.setAccessible(true);         //okay - get private field
		
		    //3-1-2) Get Cell's Value
		    String cellValue = formatter.formatCellValue(row.getCell(cellIndex));
		
		    //3-1-3) Set Cell's Value In excelData
		    field.set(excelData, cellValue);
		}
		
		excelDataInfoList.add(excelData);
	}
	
	return excelDataInfoList;
} 
```

기존의 로직과의 차이점은 ExcelDataInfo 객체의 각 필드 (ex, name, address, …)에 직접 각 Index마다의 cellValue를 넣어주지 않아도 된다. 이제 Runtime 시점에 Relection을 통해 Class의 메타데이터인 Field에 접근해 값이 주입된다. 이로써 ExcelDataInfo 클래스의 필드 변경시 해당 메서드에서 수정해야 되는 부분이 없어졌다. **(유지보수 ↑)**

<br>

**2단계)** extractDataFrom() 메서드에서 사용되는 ExcelDataInfo 타입을 제네릭 타입(<T>)으로 변경

```java
//Controller Layer
public void extractDataFromExcel(@RequestParam("excelFile") MultipartFile excelFile) throws IOException, IllegalAccessException {
    List<ExcelDataInfo> excelDataInfoList = excelService.extractDataFromV2(excelFile, ExcelDataInfo::new);
    excelDataInfoList.stream().forEach(excelDataInfo -> {log.info("excelDataInfo = {}",excelDataInfo);});
}
```

```java
//Service Layer
public <T> List<T> extractDataFrom(MultipartFile multipartFile, Supplier<T> dataSupplier) throws IOException, IllegalAccessException {  //T excelData

	//1) CREATE Workbook (==XSSFWorkbook)
	Workbook workbook = WorkbookFactory.create(multipartFile.getInputStream());

	//2) GET Sheet
	Sheet worksheet = workbook.getSheetAt(0);

	//3) EXTRACT Data
	List<T> excelDataInfoList = new ArrayList<>();

	//3-1) Extract cellValue by row in excel
	for (int rowIndex = 1; rowIndex <= worksheet.getLastRowNum(); rowIndex++) {  

	    Row row = worksheet.getRow(rowIndex);
	    if (row == null) {
		    continue;
	    }
		
	    //ExcelDataInfo excelData = ExcelDataInfo.builder().build();
	    T excelData = dataSupplier.get();
	    Field[] fields = excelData.getClass().getDeclaredFields();
		
	    for (int cellIndex = 0; cellIndex < fields.length; cellIndex++) {
	        //3-1-1) Get ExcelDataInfo's Field
	        Field field = fields[cellIndex];
	        field.setAccessible(true);         //okay - get private field
    
	        //3-1-2) Get Cell's Value
	        String cellValue = formatter.formatCellValue(row.getCell(cellIndex));
		
	        //3-1-3) Set Cell's Value In excelData
	        field.set(excelData, cellValue);
	    }
		
	    excelDataInfoList.add(excelData);
    }

    return excelDataInfoList;
}
```


기존의 extractDataFrom() 메서드에서 종속적이게 사용되고 있는 ExcelDataInfo 부분들을 모두 제네릭 타입 T로 변경하였다. 그리고 매개변수로는 매 반복마다 새로운 Object를 생성해주는   Supplier<T> dataSupplier를 추가하였다. 그 이유는  T excelData가 매개변수로 들어오게 되었을떄 반복문마다 객체의 값이 중복되는 문제가 발생하기 때문이다. 이로써 ExcelDataInfo에 의존적이었던 extractDataFrom() 메서드가 어떤 Object에도 대응하여 ExcelFile 데이터를 받을 수 있게 되었다. (코드 재사용성  ↑)


<br>

### 정리

Excel 파일과 CSV 파일을 비교해보고 대중적으로 사용되는 Excel 파일의 데이터를 추출해보는 실습을 진행해 보았다.
관련 라이브러리로는 오픈소스인 Apache POI와 OOXML 기반의 XSSF 포맷을 사용하였다.
Excel 데이터 추출시 고려해야할 점으로는 Cell과 Row가 Null일때 어떻게 처리를 해주냐였다.

Cell이 비어있을때에는 Null 값을 ‘ ‘로 치환해주었고, Row가 비어있을때에는 if-continue를 통해 NullPointerException을 해결할 수 있었다.

그리고 더 나아가서 추출한 데이터를 저장할때 특정 Object (ex, ExcelDataInfo)에 종속적인 문제를 해결하기 위해 Reflection과 Generic Type을 사용해 어떤 Object에도 대응하여 데이터를 주입할 수 있게 되었다.. 이로써 유지보수에도 이점이 생겼고, 코드 재사용이 가능해졌다.

>*전체 소스코드는 여기를 참고하면 된다.*  
>*[https://github.com/gusdn7142/Extract_Excel_Data](https://github.com/gusdn7142/Extract_Excel_Data)*


<br>


### 참고

---

- CSV & Excel
    - [https://stackoverflow.com/questions/76726572/what-is-the-difference-between-excel-and-csv](https://stackoverflow.com/questions/76726572/what-is-the-difference-between-excel-and-csv)
    - [https://www.linkedin.com/advice/0/what-pros-cons-using-csv-vs-excel-data-validation-analysis](https://www.linkedin.com/advice/0/what-pros-cons-using-csv-vs-excel-data-validation-analysis)
- Apache POI Docs
    - [https://poi.apache.org/components/spreadsheet/quick-guide.html](https://poi.apache.org/components/spreadsheet/quick-guide.html)
- Office Open XML
    - [http://officeopenxml.com/](http://officeopenxml.com/)
- Maven Repository
    - [https://mvnrepository.com/search?q=Apache+POI](https://mvnrepository.com/search?q=Apache+POI)