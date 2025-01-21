# Adobe Personalization Decisions Example

This README explains how to fetch and render personalized offers (text or images) from an Adobe personalization endpoint (or any similar personalization service) and display them on a web page.

## Table of Contents

1. [Overview](#overview)  
2. [Prerequisites](#prerequisites)  
3. [Usage](#usage)  
   - [Sample Personalization Payload](#sample-personalization-payload)  
   - [JavaScript Implementation](#javascript-implementation)  
   - [Filtering by Activity Name](#filtering-by-activity-name)  
   - [Rendering to the DOM](#rendering-to-the-dom)  
4. [Example HTML Implementation](#example-html-implementation)  
5. [Troubleshooting](#troubleshooting)

---

## Overview

The personalization engine (such as Adobe Target or Journey Optimizer) returns a JSON payload containing:

- A list of **placements** (e.g., `Main_Web_Hero_IMG`, `Main_Web_Hero_TXT`)  
- An associated **activity** (e.g., `Q19251Rec`)  
- One or more **offers** within each placement, which may include text, images, or links  

Our goal is to **extract** the relevant offer content for each placement and **insert** or **replace** it in the DOM, using plain JavaScript. This way, you can dynamically show personalized hero images, hero text, or any other personalized content on your site.

---

## Prerequisites

1. **Working Web Page**  
   You have an existing web page with elements (e.g., `<img>`, `<div>`, etc.) that you want to populate or replace with personalized content.
2. **Personalization Response**  
   You have a JSON response from Adobe personalization. This can be a local JSON object (for testing) or fetched dynamically from an endpoint.
3. **Correct CSS Selectors**  
   You know which CSS selectors (e.g., `#heroImage`) map to which placement (`Main_Web_Hero_IMG`).

---

## Usage

### Sample Personalization Payload

Below is an example JSON payload (shortened for clarity). Notice that each element in the `payload` array contains:

- **activity**: Activity metadata (e.g., `"name": "Q19251Rec"`).
- **placement**: Placement metadata (e.g., `"name": "Main_Web_Hero_TXT"`).
- **items**: One or more offers, each with `data` containing details (images, text, links, etc.).

```json
{
  "payload": [
    {
      "activity": { "name": "Q19251Rec" },
      "placement": {
        "name": "Main_Web_Hero_TXT",
        "componentType": "https://ns.adobe.com/experience/offer-management/content-component-text"
      },
      "items": [
        {
          "name": "Premium Tech Event",
          "data": {
            "format": "text/plain",
            "content": "Up to 52% off on PCs and accessories!"
          }
        }
      ]
    },
    {
      "activity": { "name": "Q19251Rec" },
      "placement": {
        "name": "Main_Web_Hero_IMG",
        "componentType": "https://ns.adobe.com/experience/offer-management/content-component-imagelink"
      },
      "items": [
        {
          "name": "ExpA Hero LOQ",
          "data": {
            "format": "image/png",
            "deliveryURL": "https://example.com/my-hero.png",
            "linkURL": "https://example.com/promo-page"
          }
        }
      ]
    }
  ]
}
```

##JavaScript Implementation
### Step 1: Extract the array of decisions (the payload)
```javascript
const { payload } = decisionsResponse;
```

### Step 2: Filter for a specific activity name
```javascript
function getDecisionsByActivityName(payload, activityName) {
  return payload.filter((entry) => entry.activity.name === activityName);
}

const myDecisions = getDecisionsByActivityName(payload, "Q19251Rec");

```
If you want all activities, skip filtering and use payload directly

### Step 3: Map placement names to your page’s CSS selectors
This is the place where you map your placements in ODE to the CSS selector you want content to be replaced by personalized content back from ODE
```javascript
const placementToSelectorMap = {
  "Main_Web_Hero_IMG": "#heroImage",
  "Main_Web_Hero_TXT": "#heroText"
};
```

### Step 4: Render the decisions to the DOM
```javascript
function renderDecisions(decisions, map) {
  decisions.forEach((decision) => {
    const { placement, items } = decision;
    const [item] = items; // assuming each placement has 1 item
    const { name: placementName, componentType } = placement;
    const selector = map[placementName];

    if (!selector) return;
    const element = document.querySelector(selector);
    if (!element) return;

    // Decide how to handle text vs. image
    if (componentType.includes("imagelink")) {
      const { deliveryURL, linkURL } = item.data;
      // If it's an <img>, we can set its src:
      element.src = deliveryURL;
      // Potentially set link if there's a parent <a> or something similar

    } else if (componentType.includes("text")) {
      const { content } = item.data;
      element.textContent = content;
    }
  });
}
```

### Step 5: Filtering by Activity Name
In your response, the activity name is stored at decisions.activity.name. If you only want decisions from a specific activity, call:
```javascript
const myDecisions = getDecisionsByActivityName(payload, "Q19251Rec");
renderDecisions(myDecisions, placementToSelectorMap);
```
If you pass an offer name instead of an activity name, you won’t get any matches (unless they happen to be the same). Double-check whether you’re filtering by the correct field.

### Step 6: Rendering to the DOM
- Text-based Offers
  - The code sets .textContent to the returned content.
- Image-based Offers
  - The code sets .src to the returned deliveryURL.
  - You can also wrap the image in a link by targeting a parent <a> element, e.g., document.querySelector("#heroLink").href = linkURL.
 
### Example (HTML Implementation)
Here’s a complete example you can open in a browser:
``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Adobe Decision Rendering Demo</title>
    <style>
      #heroImage {
        width: 400px;
        height: auto;
      }
    </style>
  </head>
  <body>
    <h1>Home Page</h1>

    <!-- Image placement -->
    <img id="heroImage" alt="Hero Offer" />

    <!-- Text placement -->
    <div id="heroText">Default text</div>

    <script>
      // 1) This is your decisions JSON (mocked for demo)
      const decisionsResponse = {
        payload: [
          {
            activity: { name: "Q19251Rec" },
            placement: {
              name: "Main_Web_Hero_TXT",
              componentType: "https://ns.adobe.com/experience/offer-management/content-component-text"
            },
            items: [
              {
                name: "Premium Tech Event",
                data: {
                  format: "text/plain",
                  content: "Up to 52% off on PCs and accessories!"
                }
              }
            ]
          },
          {
            activity: { name: "Q19251Rec" },
            placement: {
              name: "Main_Web_Hero_IMG",
              componentType: "https://ns.adobe.com/experience/offer-management/content-component-imagelink"
            },
            items: [
              {
                name: "ExpA Hero LOQ",
                data: {
                  format: "image/png",
                  deliveryURL: "https://placekitten.com/400/200",
                  linkURL: "https://example.com/promo"
                }
              }
            ]
          }
        ]
      };

      // 2) Extract payload
      const { payload } = decisionsResponse;

      // 3) Filter by activity name
      function getDecisionsByActivityName(arr, activityName) {
        return arr.filter(entry => entry.activity.name === activityName);
      }
      const targetActivityName = "Q19251Rec";
      const myDecisions = getDecisionsByActivityName(payload, targetActivityName);

      // 4) Define mapping of placement -> CSS selector
      const placementToSelectorMap = {
        "Main_Web_Hero_TXT": "#heroText",
        "Main_Web_Hero_IMG": "#heroImage"
      };

      // 5) Render function
      function renderDecisions(decisions, map) {
        decisions.forEach(decision => {
          const { placement, items } = decision;
          const [item] = items;
          const { name: placementName, componentType } = placement;

          const selector = map[placementName];
          if (!selector) return;

          const element = document.querySelector(selector);
          if (!element) return;

          if (componentType.includes("imagelink")) {
            // If it's an image
            const { deliveryURL, linkURL } = item.data;
            element.src = deliveryURL;
            // If you'd like to set a link, find a parent <a> or something similar
          } else if (componentType.includes("text")) {
            // If it's text
            const { content } = item.data;
            element.textContent = content;
          }
        });
      }

      // 6) Render them
      renderDecisions(myDecisions, placementToSelectorMap);
    </script>
  </body>
</html>

```
## Troubleshooting
+ Nothing Renders
  - Check if your CSS selector actually matches an element on page.
  - Verify the code is running after the DOM has fully loaded.
  - Confirm you are filtering the correct activity name.
+ Filtering Issues
  - Make sure you are filtering by entry.activity.name === "..." if you want the activity.
  - If you want to filter by offer name, you need to look inside entry.items[i].name.
+ Multiple items per placement
  - If "items" has more than one item, you may need to handle them in a loop, or select by priority.
+ Link Not Applied
  - If you want to wrap an image with a link, ensure you have an "<a>" element in your HTML and set its "href" prop. Alternatively, handle click on the "img" element directly.
