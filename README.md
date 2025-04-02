# Senior Phishing Alert App: A Shiny App for Phishing Prevention

## Repo Name: **senior-phishing-alert-app**

### Table of Contents:
- [Introduction](#introduction)
- [The Problem](#the-problem)
- [Case Study Objective](#case-study-objective)
- [Shiny App Features](#shiny-app-features)
- [Google Safe Browsing API Integration](#google-safe-browsing-api-integration)
- [Code](#code)
    - [Shiny App Code](#shiny-app-code)
    - [2D Heat Map Code](#2d-heat-map-code)
    - [3D Plot Code](#3d-plot-code)
- [Setting Up Google Safe Browsing API](#setting-up-google-safe-browsing-api)
- [Why This Matters](#why-this-matters)
- [Conclusion](#conclusion)
- [Future Enhancements](#future-enhancements)

---

## Introduction

In this case study, we built a **Shiny app** in **RStudio** that integrates with **Google's Safe Browsing API** to detect phishing threats targeting senior citizens. The app allows users to enter a URL and check whether it is flagged as a phishing or malware threat. Through this app, we aim to raise awareness and provide a tool that can potentially prevent financial fraud and social engineering attacks, which are increasingly affecting the elderly population.

---

## The Problem

Seniors aged 65+ are particularly vulnerable to **phishing**, **fraud**, and **social engineering attacks**, including common scams like the **Grandparent Scam**, **Tech Support Scam**, and **Financial Scam**. These scams prey on the trust and emotions of older adults, leading to devastating financial losses.

The problem is further exacerbated by the lack of technological literacy among seniors, making them easy targets for malicious actors. In 2023 alone, **seniors lost over $3.4 billion** to scams and fraud, with many victims unable to recoup their losses.

---

## Case Study Objective

The objective of this case study was to develop a **Shiny app** that:
- **Integrates with Google Safe Browsing API** to check whether a URL is safe or potentially malicious.
- **Visualizes** phishing threat data using a **2D heat map** and a **3D plot** to help identify geographic areas most vulnerable to phishing scams.
- **Educates** senior citizens and their caregivers by providing a tool that can be used to assess the safety of online links in real-time.

---

## Shiny App Features

The Shiny app developed in this case study features:
- **URL Checking**: Users can input a URL, and the app will use the Google Safe Browsing API to determine if the URL is safe or flagged as phishing/malware.
- **Interactive 2D Heat Map**: A map visualizing which areas of the U.S. have the highest vulnerability to phishing attacks based on the data.
- **Interactive 3D Plot**: A 3D plot representing vulnerability by location, showing a dynamic view of phishing threat distribution.
- **User-Friendly Interface**: Clean and simple UI allowing easy access for users of all ages.

---

## Google Safe Browsing API Integration

The **Google Safe Browsing API** allows us to programmatically check whether a URL is flagged as a phishing or malware threat. This API is instrumental in protecting seniors from potential scams while browsing the web. By using Google’s threat intelligence, we are able to keep seniors safer in real-time.

We integrated this API into our Shiny app by sending requests to Google's **Safe Browsing API** whenever a user inputs a URL, and the app responds with a message indicating whether the URL is safe or suspicious.

---

## Code

### Shiny App Code

```r
# Load required packages
library(shiny)
library(httr)
library(jsonlite)
library(leaflet)
library(ggplot2)

# API key for Google Safe Browsing API (replace with your actual key)
api_key <- "YOUR_API_KEY"

# API URL for Safe Browsing API
api_url <- "https://safebrowsing.googleapis.com/v4/threatMatches:find"

# Define UI for application
ui <- fluidPage(
    titlePanel("Senior Phishing Alert App Simulation"),
    
    sidebarLayout(
        sidebarPanel(
            textInput("urlInput", "Enter URL to Check for Phishing:", ""),
            actionButton("checkBtn", "Check URL"),
            br(),
            verbatimTextOutput("checkResult")
        ),
        
        mainPanel(
            leafletOutput("map"),
            plotOutput("barPlot")
        )
    )
)

# Define server logic
server <- function(input, output) {
    
    # Function to check URL using Google Safe Browsing API
    check_url <- function(url) {
        request_body <- list(
            client = list(
                clientId = "YourClientID",
                clientVersion = "1.0"
            ),
            threatInfo = list(
                threatTypes = c("MALWARE", "SOCIAL_ENGINEERING"),
                platformTypes = c("ANY_PLATFORM"),
                threatEntryTypes = c("URL"),
                threatEntries = list(list(url = url))
            )
        )
        
        # Send the request to the Google Safe Browsing API
        response <- POST(api_url, body = request_body, query = list(key = api_key), encode = "json")
        result <- content(response, "parsed")
        
        # Check if the URL is flagged as a threat
        if (length(result$matches) > 0) {
            return("This URL is a phishing or malware threat!")
        } else {
            return("This URL is safe.")
        }
    }
    
    # Reactive function to check the URL when the button is clicked
    observeEvent(input$checkBtn, {
        url <- input$urlInput
        if (url != "") {
            result <- check_url(url)
            output$checkResult <- renderText({ result })
        } else {
            output$checkResult <- renderText({ "Please enter a URL to check." })
        }
    })
    
    # Example leaflet map for visualizing data (you can update this with real data)
    output$map <- renderLeaflet({
        leaflet() %>%
            addTiles() %>%
            addMarkers(lng = -93.65, lat = 42.0285, popup = "Senior Population in Area")
    })
    
    # Example bar plot for phishing data (this is also a placeholder for real data)
    output$barPlot <- renderPlot({
        phishing_data <- data.frame(
            Scam_Type = c("Grandparent Scam", "Tech Support Scam", "Financial Scam"),
            Cases_Reported = c(120, 200, 180)
        )
        ggplot(phishing_data, aes(x = Scam_Type, y = Cases_Reported, fill = Scam_Type)) +
            geom_bar(stat = "identity") +
            theme_minimal() +
            labs(title = "Top Scams Targeting Seniors")
    })
}

# Run the application
shinyApp(ui = ui, server = server)
```

---

### 2D Heat Map Code

```r
# 2D Heat Map using Leaflet to show vulnerable areas
library(leaflet)

# Sample data for vulnerability (replace with real data)
senior_vulnerability_data <- data.frame(
  State = c("California", "Texas", "Florida", "New York", "Ohio"),
  Grandparent_Scam = c(120, 90, 150, 80, 60),
  Lat = c(36.7783, 31.9686, 27.9944, 40.7128, 41.2033),
  Lon = c(-119.4179, -99.9018, -81.7603, -74.0060, -82.9071)
)

# Create Leaflet Map with heat plots
leaflet(senior_vulnerability_data) %>%
  addTiles() %>%
  addCircles(lng = ~Lon, lat = ~Lat, weight = 1, radius = ~Grandparent_Scam * 10,
             color = "red", fillOpacity = 0.7) %>%
  addLegend(position = "bottomright", pal = colorNumeric("YlOrRd", senior_vulnerability_data$Grandparent_Scam), 
            values = senior_vulnerability_data$Grandparent_Scam, title = "Grandparent Scam Vulnerability")
```

---

### 3D Plot Code

```r
library(plotly)

# Create 3D plot for senior vulnerability data
plot_ly(senior_vulnerability_data, x = ~State, y = ~Grandparent_Scam, z = ~Lon, type = "scatter3d",
        mode = "markers", marker = list(size = 5, color = ~Grandparent_Scam, colorscale = 'Viridis')) %>%
  layout(title = "Senior Vulnerability to Phishing",
         scene = list(xaxis = list(title = 'State'),
                      yaxis = list(title = 'Vulnerability Percentage'),
                      zaxis = list(title = 'Longitude')))
```

---

## Setting Up Google Safe Browsing API

### Steps to get your **API Key**:
1. **Go to Google Cloud Console**: [Google Cloud Console](https://console.cloud.google.com/)
2. **Create a Project**: Click on the **Create Project** button and follow the prompts.
3. **Enable Safe Browsing API**:
   - Go to **APIs & Services > Library**.
   - Search for **Safe Browsing API** and enable it for your project.
4. **Get your API Key**:
   - Go to **APIs & Services > Credentials**.
   - Click **Create Credentials** and select **API Key**.
   - Copy the generated **API Key** and use it in the code.

---

## Why This Matters

Seniors are among the most vulnerable populations when

 it comes to online scams and phishing attacks. By integrating the Google Safe Browsing API, we can create a practical tool that allows them to safely navigate the internet and avoid potential threats. This app not only detects phishing URLs but also provides valuable insights into the geographical distribution of phishing attacks.

---

## Conclusion

In this case study, we successfully created an **interactive Shiny app** that integrates **Google's Safe Browsing API** to identify potential phishing URLs. We also visualized the data with a **2D heat map** and **3D plot**, allowing users to gain a better understanding of phishing threats. This app serves as a valuable resource for **senior citizens** and their caregivers in safeguarding their online activities.

---

## Future Enhancements

- **Real-Time Data Updates**: Integrate real-time data on phishing threats from various sources.
- **User Authentication**: Add authentication to personalize the user experience (e.g., tracking history of phishing checks).
- **Multi-Language Support**: Expand accessibility by adding support for multiple languages.
- **Mobile App Version**: Develop a mobile app version to extend the reach and usability of the tool.

---

This readme offers an in-depth overview of your project, with clear sections, detailed explanations, and all the necessary steps to set up and run the application.

