# Zeotap_Assignment - Pritish Pawar
# Application 1: Rule Engine with AST
1.	Backend (Spring Boot - Java)

# 1.1. Dependencies in pom.xml (for Maven)

<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    
    <!-- MySQL Driver -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

# 1.2. Defining AST Structure in Java
public class Node {
    private String type;      
    private Node left;        
    private Node right;       
    private String value;     

  
    public Node(String type, Node left, Node right) {
        this.type = type;
        this.left = left;
        this.right = right;
    }

    
    public Node(String type, String value) {
        this.type = type;
        this.value = value;
    }

}

# 1.3. Rule Service
import org.springframework.stereotype.Service;

@Service
public class RuleEngineService {

    
    public Node createRule(String ruleString) {
        
        Node ageNode = new Node("operand", "age > 30");
        Node deptNode = new Node("operand", "department = 'Sales'");
        Node root = new Node("operator", ageNode, deptNode);
        return root;
    }

    // Combine multiple rules into a single AST
    public Node combineRules(Node[] rules) {
        // Combine rules using AND/OR operators (example combines with AND)
        Node combinedRoot = new Node("operator", rules[0], rules[1]);
        return combinedRoot;
    }

    // Evaluate a rule against provided user data
    public boolean evaluateRule(Node root, Map<String, Object> userData) {
        if (root == null) return false;

        // Evaluate operand
        if (root.getType().equals("operand")) {
            String[] parts = root.getValue().split(" ");
            String field = parts[0];
            String operator = parts[1];
            String value = parts[2];

            // Perform comparison
            if (field.equals("age")) {
                int userAge = (int) userData.get("age");
                int conditionValue = Integer.parseInt(value);
                if (operator.equals(">")) return userAge > conditionValue;
            }
            // Handle more conditions (department, salary, etc.)
        }

        // Evaluate operator
        if (root.getType().equals("operator")) {
            if (root.getValue().equals("AND")) {
                return evaluateRule(root.getLeft(), userData) && evaluateRule(root.getRight(), userData);
            }
            if (root.getValue().equals("OR")) {
                return evaluateRule(root.getLeft(), userData) || evaluateRule(root.getRight(), userData);
            }
        }

        return false;
    }
}
# 1.4. Controller (REST API)

import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/rules")
public class RuleEngineController {
    private final RuleEngineService ruleEngineService;

    public RuleEngineController(RuleEngineService ruleEngineService) {
        this.ruleEngineService = ruleEngineService;
    }

    @PostMapping("/create")
    public Node createRule(@RequestBody String ruleString) {
        return ruleEngineService.createRule(ruleString);
    }

    @PostMapping("/combine")
    public Node combineRules(@RequestBody Node[] rules) {
        return ruleEngineService.combineRules(rules);
    }

    @PostMapping("/evaluate")
    public boolean evaluateRule(@RequestBody Map<String, Object> userData) {
        
        Node ageNode = new Node("operand", "age > 30");
        Node deptNode = new Node("operand", "department = 'Sales'");
        Node root = new Node("operator", ageNode, deptNode);

        return ruleEngineService.evaluateRule(root, userData);
    }
}
# 1.5. Database Schema 
CREATE TABLE rules (
    id INT AUTO_INCREMENT PRIMARY KEY,
    rule_string TEXT,
    ast_structure TEXT --Can store the AST in JSON format
);
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Application 2: Real-Time Weather Data Processing
Steps for Weather Monitoring Application:

1.	Backend (Spring Boot - Java)

# 1.1. Dependencies in pom.xml

<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <!-- Spring Web (for REST API calls) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- MongoDB Driver (optional if using MongoDB) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

    <!-- RestTemplate for API calls -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

# 1.2. Service to Call OpenWeatherMap API

import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class WeatherService {
    private final String API_KEY = "your_openweathermap_api_key";
    private final String BASE_URL = "http://api.openweathermap.org/data/2.5/weather?q=";

    public WeatherData getWeather(String city) {
        RestTemplate restTemplate = new RestTemplate();
        String url = BASE_URL + city + "&appid=" + API_KEY;
        return restTemplate.getForObject(url, WeatherData.class);
    }

    // Convert Kelvin to Celsius
    public double convertToCelsius(double kelvinTemp) {
        return kelvinTemp - 273.15;
    }
}

# 1.3. Weather Data Model
public class WeatherData {
    private Main main;
    private long dt;
    private String name;

    // getters and setters

    public static class Main {
        private double temp;
        private double feels_like;

       
    }
}

# 1.4. Controller for Weather Data

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/weather")
public class WeatherController {

    private final WeatherService weatherService;

    public WeatherController(WeatherService weatherService) {
        this.weatherService = weatherService;
    }

    @GetMapping("/{city}")
    public WeatherData getWeather(@PathVariable String city) {
        WeatherData data = weatherService.getWeather(city);
        data.getMain().setTemp(weatherService.convertToCelsius(data.getMain().getTemp()));
        data.getMain().setFeels_like(weatherService.convertToCelsius(data.getMain().getFeels_like()));
        return data;
    }
}

# 1.5. Aggregation and Rollups Service

import java.util.List;

@Service
public class WeatherAggregationService {

    public double calculateAverageTemperature(List<Double> temperatures) {
        return temperatures.stream().mapToDouble(Double::doubleValue).average().orElse(0.0);
    }

    public double calculateMaxTemperature(List<Double> temperatures) {
        return temperatures.stream().mapToDouble(Double::doubleValue).max().orElse(0.0);
    }

    public double calculateMinTemperature(List<Double> temperatures) {
        return temperatures.stream().mapToDouble(Double::doubleValue).min().orElse(0.0);
    }

    public String getDominantWeatherCondition(List<String> conditions) {
        return conditions.stream()
                .reduce((a, b) -> a.equals(b) ? a : "Variable").orElse("Unknown");
    }
}



