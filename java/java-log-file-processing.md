# Java log file processing

```
xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>


import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.*;
import java.nio.file.*;
import java.util.stream.Stream;

public class LogParser {
    private static final ObjectMapper mapper = new ObjectMapper();

    public static void main(String[] args) {
        Path startDir = Paths.get("./logs"); // Your log folder
        String outputFile = "filtered_results.csv";

        try (PrintWriter writer = new PrintWriter(new FileWriter(outputFile))) {
            // Write CSV Header
            writer.println("name,duration");

            // Recursively walk through directories
            try (Stream<Path> paths = Files.walk(startDir)) {
                paths.filter(Files::isRegularFile)
                     .filter(path -> path.toString().endsWith(".log"))
                     .forEach(path -> processLogFile(path, writer));
            }
            System.out.println("Processing complete. Results saved to " + outputFile);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void processLogFile(Path path, PrintWriter writer) {
        try (BufferedReader br = Files.newBufferedReader(path)) {
            String line;
            while ((line = br.readLine()) != null) {
                try {
                    JsonNode node = mapper.readTree(line);
                    if (node.has("duration") && node.get("duration").asInt() > 3) {
                        String name = node.has("name") ? node.get("name").asText() : "";
                        int duration = node.get("duration").asInt();
                        
                        // Write to CSV (simplified, assumes no commas in 'name')
                        writer.printf("%s,%d%n", name, duration);
                    }
                } catch (Exception e) {
                    // Skip lines that aren't valid JSON
                }
            }
        } catch (IOException e) {
            System.err.println("Could not read file: " + path);
        }
    }
}


```

