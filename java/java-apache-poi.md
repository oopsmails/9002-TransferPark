# Java Apache Poi

## pom

```

<dependencies>
    <!-- Apache POI for .xlsx support -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>
    <!-- Jackson for JSON parsing -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.2</version>
    </dependency>
</dependencies>


```

## Code

```
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.*;

public class JsonToExcel {
    public static void main(String[] args) throws IOException {
        String inputPath = "data.txt";   // Your JSON lines file
        String outputPath = "output.xlsx";
        
        ObjectMapper mapper = new ObjectMapper();
        try (Workbook workbook = new XSSFWorkbook();
             BufferedReader reader = new BufferedReader(new FileReader(inputPath))) {
            
            Sheet sheet = workbook.createSheet("JSON Data");
            String line;
            int rowNum = 0;
            List<String> headers = new ArrayList<>();

            while ((line = reader.readLine()) != null) {
                JsonNode jsonNode = mapper.readTree(line);
                
                // Create Header Row on the first line
                if (rowNum == 0) {
                    Row headerRow = sheet.createRow(rowNum++);
                    jsonNode.fieldNames().forEachRemaining(headers::add);
                    for (int i = 0; i < headers.size(); i++) {
                        headerRow.createCell(i).setCellValue(headers.get(i));
                    }
                }

                // Create Data Rows
                Row row = sheet.createRow(rowNum++);
                for (int i = 0; i < headers.size(); i++) {
                    JsonNode value = jsonNode.get(headers.get(i));
                    row.createCell(i).setCellValue(value != null ? value.asText() : "");
                }
            }

            // Write to file
            try (FileOutputStream fileOut = new FileOutputStream(outputPath)) {
                workbook.write(fileOut);
            }
            System.out.println("Excel file created successfully!");
        }
    }
}
```

