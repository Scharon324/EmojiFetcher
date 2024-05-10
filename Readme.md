## Challenge
Java Developer Test Assignment

High level description:

Write a Java application that uses a http client to fetch three random emojis from the emoji API
and displays the resulting three emojis in the console. In case of failure (any kind), fall back to
sad face emoji.

Test the code for two cases using a mocked API (either mockito or custom implementation):
1. Successful responses
2. Network failure

Usage of any external library is totally okay and can be done without confirmation. Can use any
build system (gradle or maven or just compiled java).
Expected outcome:

1. Java project with instructions on how to run and test the code.
2. Documentation/reference about the libraries used (if any)

Resources:
Random Emoji API endpoint: https://emojihub.yurace.pro/api/random

## Step by step guide how i did it.

Dependency management is a core feature of Maven. Managing dependencies for a single project is easy. Managing dependencies for multi-module projects and applications that consist of hundreds of modules is possible. Maven helps a great deal in defining, creating, and maintaining reproducible builds with well-defined classpaths and library versions. „ 

# Import

This scope is only supported on a dependency of type pom in the <dependencyManagement> section. It indicates the dependency is to be replaced with the effective list of dependencies in the specified POM's <dependencyManagement> section. Since they are replaced, dependencies with a scope of import do not actually participate in limiting the transitivity of a dependency.
#
Received following error:

### <dependency>
    Exception in thread "main" java.lang.Error: Unresolved compilation problems: JSONArray cannot be resolved to a type JSONObject cannot be resolved to a type JSONObject cannot be resolved to a type 
    at com.example.EmojiFetcher5.EmojiFetcher.main(EmojiFetcher.java:29).
</dependency>

#

While researching looks like the error I have encountered indicates that the classes JSONArray and JSONObject are not being recognized by the compiler. These classes are part of the JSON-java library, which is not included in my project dependencies.

To resolve this issue, I need to add the JSON-java library to my Maven project. I will just add the following dependency to our pom.xml file: 


### <dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20210307</version>
</dependency>

This dependency will ensure that the JSON-java library is downloaded and included in my project, allowing me to use classes like JSONArray and JSONObject.
After running the code again:
I'm getting following error: 

##

### <dependency>
    Exception in thread "main" org.json.JSONException: A JSONObject text must begin with '{' at 1 [character 2 line 1] at org.json.JSONTokener.syntaxError(JSONTokener.java:507)  at org.json.JSONObject.<init>(JSONObject.java:222)  at org.json.JSONObject.<init>(JSONObject.java:406)  at com.example.EmojiFetcher5.EmojiFetcher.main(EmojiFetcher.java:29)
</dependency>

After research the error I have encountered indicates that the response I’m receiving from the emoji API is not in the expected JSON format. The JSONObject constructor expects the input text to begin with '{' to indicate the start of a JSON object, but it seems like the response is not formatted correctly.

To resolve this issue, I’ll make sure that I’m receiving the response from the API correctly. I can print out the response body to see what exactly I am getting. I have added the following code to do that:

    System.out.println("Response Body: " + jsonResponse);

Adding this line before trying to parse the JSON response. This will help to understand the format of the response and identify any issues with it.

After running the code again:

    Response Body: PNG  

    Exception in thread "main" org.json.JSONException: A JSONObject text must begin with '{' at 1 [character 2 line 1]
    at org.json.JSONTokener.syntaxError(JSONTokener.java:507) 
    at org.json.JSONObject.<init>(JSONObject.java:222) 
    at org.json.JSONObject.<init>(JSONObject.java:406) 
    at com.example.EmojiFetcher5.EmojiFetcher.main(EmojiFetcher.java:30)


After investigation It appears that the response is not in JSON format instead, it seems to be a PNG image. This explains why the JSON parser is failing when trying to parse it into a JSONObject.

The issue likely stems from the incorrect URL (Which I used different URL to try and test it out instead oops) or endpoint being used to fetch the random emojis. The API endpoint I was currently using might not be returning the expected JSON response.

To resolve this issue, I have changed the correct URL endpoint for fetching random emojis. Once I have the correct URL, I have updated the API_URL variable in my code accordingly. So, I have just updated the following line of code:

    private static final String API_URL = "THIS WAS DIFFERENT BECAUSE I USED DIFFERENT URL INSTEAD OF WHAT WAS GIVEN xD";

After running the code again I’m getting a new error:

    Response Body: {"name":"french fries","category":"food and drink","group":"food prepared","htmlCode":["\u0026#127839;"],"unicode":["U+1F35F"]} Exception in thread "main" org.json.JSONException: JSONObject["emojis"] not found. 
    at org.json.JSONObject.get(JSONObject.java:572) 
    at org.json.JSONObject.getJSONArray(JSONObject.java:765) 
    at com.example.EmojiFetcher5.EmojiFetcher.main(EmojiFetcher.java:30)

Note:

    I am still using ("Response Body: " + jsonResponse); Code to see what response I’m receiving from the API. System.out.println("Response Body: " + jsonResponse);

After researching above error, I can say that the response is indeed in JSON format, but it has a different structure than what the code expects. The error message indicates that the key "emojis" is not found in the JSON object.

Based on the response body provided by the console/terminal, it seems that the array of emojis is nested within another object with keys like "name", "category", "group", etc.
To fix this issue, I will need to adjust my code to access the correct nested array of emojis. So, I have modified my code:

    public class EmojiFetcher5 {
    private static final String API_URL = "https://emojihub.yurace.pro/api/random";
    public static void main(String[] args) {
        HttpClient httpClient = HttpClients.createDefault();
        HttpGet request = new HttpGet(API_URL);
        try {
            HttpResponse response = httpClient.execute(request);
            String jsonResponse = EntityUtils.toString(response.getEntity());

            JSONObject jsonObject = new JSONObject(jsonResponse);
            JSONArray emojisArray = jsonObject.getJSONArray("htmlCode");

            Random random = new Random();
            for (int i = 0; i < 3; i++) {
                String emoji = emojisArray.getString(random.nextInt(emojisArray.length()));
                System.out.println("Emoji " + (i+1) + ": " + emoji);
            }

        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

In this modified code:

I have first parse the entire JSON response into a JSONObject. Then, I extract the array of emojis using the key "htmlCode" and I iterate over this array to retrieve and print three random emojis.

The result outcome in the terminal/console:

    Emoji 1: &#127997; 
    Emoji 2: &#129332; 
    Emoji 3: &#127997;

While researching I have found out that emojis are being retrieved as HTML encoded entities (e.g., "") instead of their actual emoji representations. This is because the API is providing the emojis as HTML codes, and the code is printing them directly without decoding them.

In order for me to fix this issue, I will need to decode the HTML entities into their corresponding characters before printing them. I can achieve this using the StringEscapeUtils class from the Apache Commons Text library. First, I will need to add the Apache Commons Text library as a dependency in my Maven project.

I need to add following dependency to my pom.xml file as follows:

    <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.9</version>
    </dependency>

Then, I need to modify my Java code to decode the HTML entities before printing the emojis. Result:

    import org.apache.commons.text.StringEscapeUtils;

    public class EmojiFetcher5 {

    private static final String API_URL = "https://emojihub.yurace.pro/api/random";

    public static void main(String[] args) {
        HttpClient httpClient = HttpClients.createDefault();
        HttpGet request = new HttpGet(API_URL);

        try {
            HttpResponse response = httpClient.execute(request);
            String jsonResponse = EntityUtils.toString(response.getEntity());

            JSONObject jsonObject = new JSONObject(jsonResponse);
            JSONArray emojisArray = jsonObject.getJSONArray("htmlCode");

            Random random = new Random();
            for (int i = 0; i < 3; i++) {
                String emojiCode = emojisArray.getString(random.nextInt(emojisArray.length()));
                String emoji = StringEscapeUtils.unescapeHtml4(emojiCode); // Decode HTML entity
                System.out.println("Emoji " + (i+1) + ": " + emoji);
            }

        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    }

With this code change, the HTML encoded entities will be decoded into their corresponding emoji characters before printing them to the console.

Result in the following console: 

    Emoji 1: ⃣ 
    Emoji 2: ️ 
    Emoji 3: ️

While researching it seems like the emojis are still not being displayed correctly. This might be due to the fact that the emojis are represented by surrogate pairs or other Unicode characters that are not properly handled by the StringEscapeUtils.unescapeHtml4() method.

To handle emojis correctly, I can try a different approach by directly parsing the Unicode code points and converting them into emojis. So, I have added the below code to my already existing code:

    private static String convertHtmlCodeToEmoji(String htmlCode) {

        // Remove '&#' prefix and ';' suffix to get the Unicode code point
        String unicodeStr = htmlCode.substring(2, htmlCode.length() - 1);

        // Convert Unicode code point to int
        int unicodeInt = Integer.parseInt(unicodeStr);

        // Create String from Unicode code point
        return new String(Character.toChars(unicodeInt));

I have defined a convertHtmlCodeToEmoji() method that takes a HTML code (e.g., '') as input and converts it into the corresponding emoji. Inside this method, I extract the Unicode code point from the HTML code and convert it into an integer. Then use Character.toChars() method to convert the integer into a String representing the emoji.

When running the code I got the result as follows:

    Exception in thread "main" java.lang.Error: Unresolved compilation problem: IOException cannot be resolved to a type at com.example.EmojiFetcher5.EmojiFetcher.main(EmojiFetcher.java:45)

On my research this error is simply common that compiler is unable to resolve the type IOException. This typically occurs when the necessary import statement is missing.

To resolve this issue, I will just need to ensure that the IOException class is imported from the correct package (java.io). So, I will just add to my project EmojiFetcher5.java file that will includes the proper import statement for IOException. Just adding the following import code:

    import java.io.IOException;

### SOME PROGRESS:

Finally, when running the code I get the following result in terminal/console:
The result was: 

    Emoji 1: 🏽
    Emoji 2: 🙇
    Emoji 3: 🙇

    Emoji 1: 🏽
    Emoji 2: 🙇
    Emoji 3: 🙇

When running the code unfortunately it shows always only 1 random icon and two duplicates icons. Which I was not happy about so I continued my investigation and reviewing the code.

So, I have modified the code as follows:

    import org.apache.http.client.ClientProtocolException;
    import org.apache.http.client.HttpClient;
    import org.apache.http.client.methods.HttpGet;
    import org.apache.http.impl.client.HttpClients;
    import org.apache.http.HttpResponse;
    import org.apache.http.util.EntityUtils;
    import org.json.JSONArray;
    import org.json.JSONObject;

    import java.io.IOException;
    import java.util.HashSet;
    import java.util.Random;
    import java.util.Set;

    public class EmojiFetcher5 {

    private static final String API_URL = " https://emojihub.yurace.pro/api/random ";

    public static void main(String[] args) {
        HttpClient httpClient = HttpClients.createDefault();
        HttpGet request = new HttpGet(API_URL);

        try {
            HttpResponse response = httpClient.execute(request);
            String jsonResponse = EntityUtils.toString(response.getEntity());

            JSONObject jsonObject = new JSONObject(jsonResponse);
            JSONArray emojisArray = jsonObject.getJSONArray("htmlCode");

            Random random = new Random();
            Set<Integer> selectedIndices = new HashSet<>();
            int selectedCount = 0;
            while (selectedCount < 3) {
                int randomIndex = random.nextInt(emojisArray.length());
                if (!selectedIndices.contains(randomIndex)) {
                    String emojiCode = emojisArray.getString(randomIndex);
                    String emoji = convertHtmlCodeToEmoji(emojiCode);
                    System.out.println("Emoji " + (selectedCount + 1) + ": " + emoji);
                    selectedIndices.add(randomIndex);
                    selectedCount++;
                }
            }

        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String convertHtmlCodeToEmoji(String htmlCode) {
        // Remove '&#' prefix and ';' suffix to get the Unicode code point
        String unicodeStr = htmlCode.substring(2, htmlCode.length() - 1);
        // Convert Unicode code point to int
        int unicodeInt = Integer.parseInt(unicodeStr);
        // Create String from Unicode code point
        return new String(Character.toChars(unicodeInt));
    }
    }

I used a Set called selectedIndices to keep track of the indices of the emojis that have already been selected. I tried to ensure that it continues selecting emojis until it have selected three unique emojis. I used a while loop to repeatedly select emojis until it has reached the desired count of three unique emojis. I have tried to ensure that each displayed emoji is unique.

Result: 

    I broke it :D 

In other words, it will fetch only 2 emojis and kept running the terminal/console and never caught the third random emoji which increased a CPU process to 100%. Whoops.

Upon my research the loop isn't generating unique emojis as intended. I need to adjust the logic to ensure that code will select three distinct emojis randomly from the available options.

I can achieve this by shuffling the indices of the emojis and then selecting the first three indices from the shuffled list. This approach guarantees uniqueness without needing to repeatedly select and check.

The code I wrote is as follows:

    import org.apache.http.client.ClientProtocolException;
    import org.apache.http.client.HttpClient;
    import org.apache.http.client.methods.HttpGet;
    import org.apache.http.impl.client.HttpClients;
    import org.apache.http.HttpResponse;
    import org.apache.http.util.EntityUtils;
    import org.json.JSONArray;
    import org.json.JSONObject;

    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;

    public class EmojiFetcher5 {

    private static final String API_URL = " https://emojihub.yurace.pro/api/random ";

    public static void main(String[] args) {
        HttpClient httpClient = HttpClients.createDefault();
        HttpGet request = new HttpGet(API_URL);

        try {
            HttpResponse response = httpClient.execute(request);
            String jsonResponse = EntityUtils.toString(response.getEntity());

            JSONObject jsonObject = new JSONObject(jsonResponse);
            JSONArray emojisArray = jsonObject.getJSONArray("htmlCode");

            List<Integer> indices = new ArrayList<>();
            for (int i = 0; i < emojisArray.length(); i++) {
                indices.add(i);
            }
            Collections.shuffle(indices);

            for (int i = 0; i < 3; i++) {
                int index = indices.get(i);
                String emojiCode = emojisArray.getString(index);
                String emoji = convertHtmlCodeToEmoji(emojiCode);
                System.out.println("Emoji " + (i + 1) + ": " + emoji);
            }

        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String convertHtmlCodeToEmoji(String htmlCode) {
        // Remove '&#' prefix and ';' suffix to get the Unicode code point
        String unicodeStr = htmlCode.substring(2, htmlCode.length() - 1);
        // Convert Unicode code point to int
        int unicodeInt = Integer.parseInt(unicodeStr);
        // Create String from Unicode code point
        return new String(Character.toChars(unicodeInt));
    }
    }

What I did I have created a list of indices representing the indices of the emojis in the array. I shuffle the list of indices to randomize the order and then it will select the first three indices from the shuffled list and retrieve the corresponding emojis.

The Result outcome:

    Emoji 1: 🙆 
    Exception in thread "main" Emoji 2: 🏿 
    java.lang.IndexOutOfBoundsException: Index 2 out of bounds for length 2 
    at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64) 
    at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70) 
    at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266) 
    at java.base/java.util.Objects.checkIndex(Objects.java:361) at java.base/java.util.ArrayList.get(ArrayList.java:427) 
    at com.example.EmojiFetcher5.EmojiFetcher.main(EmojiFetcher.java:39)

Upon my investigation It seems there's an issue with the index calculation. The error indicates that the index is out of bounds, which means it's trying to access an element in the list beyond its size.

In this code below:

    for (int i = 0; i < 3; i++) {
        int index = indices.get(i);
        String emojiCode = emojisArray.getString(index);
        String emoji = convertHtmlCodeToEmoji(emojiCode);
        System.out.println("Emoji " + (i + 1) + ": " + emoji);
    }

The error occurs when the loop attempts to access the third index from the indices list, but the list only contains two indices. This happens when the number of emojis fetched from the API is less than three.
To fix this issue, I need to ensure that I only iterate over the number of emojis actually fetched from the API. I can achieve this by using the minimum of the list size and 3 as the loop condition.

So, I have modified the code to:

    int emojiCount = Math.min(3, indices.size());
    for (int i = 0; i < emojiCount; i++) {
        int index = indices.get(i);
        String emojiCode = emojisArray.getString(index);
        String emoji = convertHtmlCodeToEmoji(emojiCode);
        System.out.println("Emoji " + (i + 1) + ": " + emoji);
    }

This modification ensures that the loop iterates only up to the minimum of the number of emojis fetched and 3.

When running the code again the outcome:

    Emoji 1: 🚻

In this instance I did modify the code to give me 3 emojis but it still showed 1 emoji so my investigation and thinking continues.
In this instance I have added yet again print statement to display the jsonResponse: that will help me to identify if the issue lies in the response from the API.

Outcome of JsonResponse in the console is as follows:

    JSON Response: {"name":"rat","category":"animals and nature","group":"animal mammal","htmlCode":["\u0026#128000;"],"unicode":["U+1F400"]}

To ensure that it will always display three emojis, even if the API response contains fewer than three, ill modify the code to fetch multiple random emojis from the API until I have collected three unique emojis. This way, even if the API returns fewer than three emojis, I will still be able to make multiple requests until I have collected three unique ones. So, I have started from the beginning and reviewed my code and wrote it again with new logic implementations.

My code outcome:

    import org.apache.http.client.ClientProtocolException;
    import org.apache.http.client.HttpClient;
    import org.apache.http.client.methods.HttpGet;
    import org.apache.http.impl.client.HttpClients;
    import org.apache.http.HttpResponse;
    import org.apache.http.util.EntityUtils;
    import org.json.JSONArray;
    import org.json.JSONObject;

    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.HashSet;
    import java.util.List;
    import java.util.Random;
    import java.util.Set;

    public class EmojiFetcher5 {

    private static final String API_URL = " https://emojihub.yurace.pro/api/random ";
    private static final int MAX_EMOJIS_TO_FETCH = 3;

    public static void main(String[] args) {
        HttpClient httpClient = HttpClients.createDefault();

        List<String> emojis = new ArrayList<>();
        while (emojis.size() < MAX_EMOJIS_TO_FETCH) {
            try {
                HttpGet request = new HttpGet(API_URL);
                HttpResponse response = httpClient.execute(request);
                String jsonResponse = EntityUtils.toString(response.getEntity());

                JSONObject jsonObject = new JSONObject(jsonResponse);
                JSONArray emojisArray = jsonObject.getJSONArray("htmlCode");

                for (int i = 0; i < emojisArray.length(); i++) {
                    String emojiCode = emojisArray.getString(i);
                    emojis.add(emojiCode);
                }
            } catch (ClientProtocolException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        // Shuffle the list of emojis
        List<String> shuffledEmojis = new ArrayList<>(emojis);
        Collections.shuffle(shuffledEmojis);

        // Display the first three emojis (or all if fewer than three)
        int numEmojisToDisplay = Math.min(MAX_EMOJIS_TO_FETCH, shuffledEmojis.size());
        for (int i = 0; i < numEmojisToDisplay; i++) {
            String emojiCode = shuffledEmojis.get(i);
            String emoji = convertHtmlCodeToEmoji(emojiCode);
            System.out.println("Emoji " + (i + 1) + ": " + emoji);
        }
    }

    private static String convertHtmlCodeToEmoji(String htmlCode) {
        // Remove '&#' prefix and ';' suffix to get the Unicode code point
        String unicodeStr = htmlCode.substring(2, htmlCode.length() - 1);
        // Convert Unicode code point to int
        int unicodeInt = Integer.parseInt(unicodeStr);
        // Create String from Unicode code point
        return new String(Character.toChars(unicodeInt));
    }
    }

I made that it continuously fetches emojis from the API until I have collected three unique emojis or until I reach the maximum number of requests. Then I shuffle the list of fetched emojis to randomize their order. Then by logic it should then display the first three emojis from the shuffled list or all emojis if fewer than three were fetched.

AND FINALLY CORRECT OUTCOME:

    Emoji 1: 🛋
    Emoji 2: 🏫
    Emoji 3: 📧
    Emoji 1: 🛋
    Emoji 2: 🏫
    Emoji 3: 📧

### Project Structure and test the code –

The project utilizes the Apache HttpClient library to make HTTP requests to the emoji API and the JSON library to parse the JSON response.

Project Structure:

    EmojiFetcher/
    │
    ├── src/
    │   └── com/
    │       └── example/
    │           └── EmojiFetcher5.java
    │
    └── pom.xml

Dependencies:
Apache HttpClient: Used to make HTTP requests to the emoji API.
JSON: Used to parse the JSON response from the emoji API.

Upon running the application, it will fetch three random emojis from the emoji API and display them in the console. My project will display emojis each time I run my application, indicating that random emojis are indeed fetched from the API.

Library Documentation:

    Apache HttpClient:
    Purpose: Apache HttpClient is a powerful and flexible HTTP client library for Java. It provides support for the HTTP protocol and allows developers to make HTTP requests to web servers.
    •	Features:
    o	Ease of Use: Provides a simple and intuitive API for making HTTP requests and handling responses.
    o	Customization: Offers extensive configuration options and allows customization of request parameters, headers, and authentication methods.
    o	Connection Management: Manages HTTP connections efficiently, including connection pooling and connection timeout settings.
    o	SSL/TLS Support: Supports secure connections via SSL/TLS protocols, enabling secure communication with web servers.
    JSON:
    •	Purpose: The JSON library provides support for parsing and manipulating JSON data in Java applications. It allows developers to convert JSON strings to Java objects and vice versa, enabling seamless integration with JSON-based web services and APIs.

    •	Features:
    o	Parsing: Parses JSON strings into Java objects, including JSONObject and JSONArray.
    o	Serialization: Serializes Java objects into JSON strings.
    o	Easy to Use: Provides a simple and intuitive API for working with JSON data.
    o	Platform-Independent: Works across different Java platforms and environments.
    •	Documentation: Official documentation for the JSON library can be found below list of references.

These libraries provide essential functionality for making HTTP requests and handling JSON data in Java applications. By leveraging these libraries, developers can efficiently interact with web services and APIs that communicate using the HTTP protocol and JSON format.


My Completed Code Project are in GitHub Rep:

##

Added the following features :
1. Code reusability
2. Object-oriented programming
3. Code simplicity and organization

New code and some changes:
1. Code Reusability: Currently, the code directly fetches and processes emojis within the main method. This tightly couples the fetching logic with the application's main logic. I can improve code reusability by separating concerns. I can create a separate class responsible for fetching emojis, allowing me to reuse this logic in other parts of the application or in other projects.

2. Object-Oriented Programming: I can leverage object-oriented principles to improve the structure of my code. For example, I can create classes to represent Emojis, API clients, and HTTP responses. This helps encapsulate related functionality within classes, making the code easier to understand and maintain.

3. Code Simplicity and Organization: To improve code simplicity and organization, I can break down complex tasks into smaller, more manageable methods. This improves readability and makes it easier to understand the code's logic. Additionally, I can organize related methods and classes into packages to provide a clear structure to the project.

In the refactored version, the Main class is responsible for orchestrating the fetching and displaying of emojis. It utilizes the EmojiFetcher class to fetch emojis and then processes and displays them accordingly.

These changes improve code reusability, follow object-oriented principles, and enhance code simplicity and organization.

### References:

https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html
https://stackoverflow.com/questions/43813670/how-can-i-solve-org-json-simple-jsonobject-cannot-be-resolved#43813822
https://stackoverflow.com/questions/4773663/jsonobject-text-must-begin-with
https://github.com/taskadapter/redmine-java-api/issues/126
https://stackoverflow.com/questions/76427196/how-to-detect-if-string-contains-emoji-in-java
https://stackoverflow.com/questions/50452770/api-data-returning-unicode-characters-in-console#50663160
https://stackoverflow.com/questions/38181966/print-unicode-emoji-from-api-response
https://github.com/abourtnik/emojis-world
https://mvnrepository.com/artifact/org.apache.commons/commons-lang3
https://stackoverflow.com/questions/15919267/stringescapeutils-can-not-be-resolved
https://mvnrepository.com/artifact/org.apache.httpcomponents
https://hc.apache.org/httpcomponents-client-4.5.x/current/httpclient/dependency-info.html
https://stackoverflow.com/questions/3915961/how-to-view-hierarchical-package-structure-in-eclipse-package-explorer

