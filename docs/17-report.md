[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Report

Membuat report menggunakan library [JasperReport](https://community.jaspersoft.com/documentation/).

``` java
public interface ReportHandler {
	List<ReportType> getReportTypes();
	JasperPrint createPrint(ReportInput input) throws JRException;
	ReportResult createReport(ReportInput input, boolean useExportManager) throws Exception;
	ReportResult createReport(ReportInput input) throws Exception;
	void exportReport(ReportInput input, OutputStream outputStream) throws Exception;
}

public class ReportInput {
	private JasperReport report; 
	private ReportType type;
	private MapStringObject parameters;
	private Collection<?> datasource;
}

public class ReportResult {
	private final ReportType type;	
	private final byte[] bytes;
}
```

## Bean

``` java
@Singleton
ReportHandler reportHandler() {
    return new ReportHandlerImpl();
}
```

## Type

* PDF
* XLS
* XLSX
* RTF
* DOCX
* PPTX
* CSV
* XML
* JSON
* HTML

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
