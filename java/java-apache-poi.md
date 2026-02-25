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

- V1: including all fields

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


- V2: Custermize fields

```
You can easily control the output by defining a whitelist of field names in a specific order.
Instead of dynamically extracting headers from the first JSON line, you can define a List<String> or an Array containing only the fields you want, in the exact order you want them to appear.
Updated Code Strategy
Here is how to modify the previous logic to include only specific fields (e.g., "id", "name", and "email") in that specific order:
java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.Arrays;
import java.util.List;

public class JsonToExcelFiltered {
    public static void main(String[] args) throws IOException {
        String inputPath = "data.txt";
        String outputPath = "filtered_output.xlsx";

        // 1. Define your desired headers and their order here
        List<String> columnsToInclude = Arrays.asList("id", "name", "email", "status");

        ObjectMapper mapper = new ObjectMapper();
        try (Workbook workbook = new XSSFWorkbook();
             BufferedReader reader = new BufferedReader(new FileReader(inputPath))) {

            Sheet sheet = workbook.createSheet("Filtered Data");
            
            // 2. Create the Header Row manually based on your list
            Row headerRow = sheet.createRow(0);
            for (int i = 0; i < columnsToInclude.size(); i++) {
                headerRow.createCell(i).setCellValue(columnsToInclude.get(i));
            }

            String line;
            int rowNum = 1; // Start data at row 1

            while ((line = reader.readLine()) != null) {
                if (line.trim().isEmpty()) continue; // Skip empty lines

                JsonNode jsonNode = mapper.readTree(line);
                Row row = sheet.createRow(rowNum++);

                // 3. Only pull data for the headers you defined
                for (int i = 0; i < columnsToInclude.size(); i++) {
                    String fieldName = columnsToInclude.get(i);
                    JsonNode value = jsonNode.get(fieldName);
                    
                    // Set cell value; check for null in case the field is missing in JSON
                    String cellValue = (value == null || value.isNull()) ? "" : value.asText();
                    row.createCell(i).setCellValue(cellValue);
                }
            }

            // Auto-size columns for better readability
            for (int i = 0; i < columnsToInclude.size(); i++) {
                sheet.autoSizeColumn(i);
            }

            try (FileOutputStream fileOut = new FileOutputStream(outputPath)) {
                workbook.write(fileOut);
            }
            System.out.println("Excel file with filtered columns created!");
        }
    }
}

```


