Poi 라이브러리를 이용한 Excel download



>poi란 
>Apache 소프트웨어 재단에서 만든 라이브러리로 마이크로소프트 오피스 파일 포맷을 순수 자바 언어로서 읽고 쓰는 기능을 제공.
>주로 워드, 엑셀, 파워포인트와 파일을 지원한다. 



1. 내가 원하는 데이터를 Excel파일로 다운로드 하는방법
2. Excel파일을 읽어서 데이터를 뽑는 방법 



### 1) poi library 추가하기 (maven/gradle)

---

```
<!-- maven -->

	<!-- xls 전용 -->
	<dependency>
    	<groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>5.0.0</version>
    </dependency>

    <!-- xlsx 전용 -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.0.0</version>
     </dependency>
        
<!-- gradle -->

    implementation 'org.apache.poi:poi:5.0.0'
    implementation 'org.apache.poi:poi-ooxml:5.0.0'

```



### 2) Excel Download

---

`/resources/excelForm/test.xlsx` 여기경로에 맞게 내가 원하는 엑셀폼 파일을 넣어두고,
 그 폼을 가지고와서 내가 원하는 데이터를 가공해 넣을 수 있다.

```
public class ExcelService {
    private final int ROW_ACCESS_WINDOW_SIZE = 100;

    @Autowired
    ResourceLoader resourceLoader;

    public SXSSFWorkbook initExcelTemplate(String templagePath) throws IOException {
        InputStream templateFile = resourceLoader.getResource(templagePath).getInputStream();
        SXSSFWorkbook sxssfWorkbook = new SXSSFWorkbook(new XSSFWorkbook(templateFile), ROW_ACCESS_WINDOW_SIZE);
        return sxssfWorkbook;
    }

    public void creatRowData(Row row, Object parameterClass) throws Exception{

        Field[] fields = parameterClass.getClass().getDeclaredFields();
        int fieldLength = fields.length;

        for (int i = 0; i < fieldLength; i++) {
            Method method = parameterClass.getClass().getMethod("get" + fields[i].getName().substring(0, 1).toUpperCase() + fields[i].getName().substring(1));
            String cellValue = (String) method.invoke(parameterClass);
            row.createCell(i + 1).setCellValue(cellValue);
        }
    }

    public void creatRowDataByNotNum(Row row, Object parameterClass) throws Exception {
        Field[] fields = parameterClass.getClass().getDeclaredFields();
        int fieldLength = fields.length;

        for (int i = 0; i < fieldLength; i++) {
            Method method = parameterClass.getClass().getMethod("get" + fields[i].getName().substring(0, 1).toUpperCase() + fields[i].getName().substring(1));
            String cellValue = (String) method.invoke(parameterClass);
            row.createCell(i).setCellValue(cellValue);
        }
    }

    public void creatRowDataByHeader(Row row, Object parameterClass) throws Exception {
        Field[] fields = parameterClass.getClass().getDeclaredFields();
        int fieldLength = fields.length;

        for (int i = 0; i < fieldLength; i++) {
            Method method = parameterClass.getClass().getMethod("get" + fields[i].getName().substring(0, 1).toUpperCase() + fields[i].getName().substring(1));
            String cellValue = (String) method.invoke(parameterClass);
            row.createCell(i).setCellValue(cellValue);
        }

    }
}

```



    @GetMapping("/excel")
    public void excelDownload(ExcelSearchRequest request, HttpServletResponse response) {
    	
    	try {
        		String filename = "excel_" + StringUtil.getTodayDateString("yyyyMMddHHmmss"); 
    
                SXSSFWorkbook sxss = ExcelService.initExcelTemplate ("/resources/excelForm/test.xlsx");  
                Sheet sheet = sxss.getSheetAt(0);
                
    			//내가 가져오는 데이터 list에 담아서 가져오기 
                List<ExcelSearchResponse> responses = excelService.findByExcelSearchList(request);
    
                int rownum = 0;
                int totcount = responses.size();
                double per = 0;
    			int progress = 0;
    
                for(ExcelSearchResponse excelSearchResponse : responses) {
          
                    Row row = sheet.createRow(++rownum);
                    row.createCell(0).setCellValue(responses.getName());
                    row.createCell(1).setCellValue(responses.getAddress());
                    row.createCell(2).setCellValue(responses.getPhoneNumber());
                    
                    per = (double)rownum / (double)totcount * 100.0;
                    progress = (int)per;
                }
    
                if(totcount == 0) progress = 100;
    
                response.setContentType("application/msexcel");
                response.setHeader("Content-Disposition", "attachment; filename=" 
                					+ java.net.URLEncoder.encode(filename, "UTF-8") + ".xlsx");
                sxss.write(response.getOutputStream());
                sxss.dispose();
                
        	} catch (FileNotFoundException e) {
        		log.error(e.getMessage());
        	} catch (IOException e) {
        		log.error(e.getMessage());
        	} catch (Exception e) {
        		log.error(e.getMessage());
        	}
    }
        



### 3) Excel 파일 읽기

```
public void excelRead(Reservation reservation) {

    if (reservation.getParticipantsFile() != null) {
        String filePath = storageService.getFilePath(reservation.getParticipantsFile());

        try {
            FileInputStream file = new FileInputStream(filePath);
            XSSFWorkbook workbook = new XSSFWorkbook(file);

            XSSFSheet sheet = workbook.getSheetAt(0); // 해당 엑셀파일의 시트(Sheet) 수
            int rows = sheet.getPhysicalNumberOfRows(); // 해당 시트의 행의 개수

            List<Participant> participants = new ArrayList<>();

            for (int rowIndex = 1; rowIndex < rows; rowIndex++) {
                XSSFRow row = sheet.getRow(rowIndex); // 각 행을 읽어온다
                if (row != null) {
                    String name = row.getCell(0).toString(); //이름
                    String phone = row.getCell(1).toString(); //핸드폰
                    phone = phone.replace("-", ""); //하이픈(-)제거
                    String grade = row.getCell(2).toString(); //학년

                    if (StringUtils.isNotEmpty(name) && StringUtils.isNotEmpty(phone)) {
                        Participant participant = new Participant();
                        participant.setReservation(reservation);
                        participant.setEduCourse(reservation.getEduCourse());
                        participant.setName(name);
                        participant.setPhoneNumber(phone);
                        participant.setDeleted(false);
                        participants.add(participant);
                    }
                }
            }

            log.debug("엑셀파일에서 참가자 인원 정보를 저장합니다.");
            participantRepository.saveAll(participants);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```

 