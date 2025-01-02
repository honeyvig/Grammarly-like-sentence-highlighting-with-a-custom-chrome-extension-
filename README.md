# Grammarly-like-sentence-highlighting-with-a-custom-chrome-extension
Please see it add https://github.com/safekids-ai/ml-models/tree/main/apps/gmail-extension and try it at

https://chromewebstore.google.com/detail/safe-kids-ai-for-email/pnaedlhkmadjdgjkenjmaepfaiiioocc?hl=en-US.

My goal is to make the highlighting piece very similar to Grammarly. The sentence is highlighted and can be Dismissed.
---------
To enhance your Chrome extension and make the sentence highlighting feature similar to Grammarly, you would need to implement the following changes to the current functionality of your extension:

    Highlighting Text: Detect and highlight sentences in the email content.
    Dismiss Action: Allow users to dismiss the highlighted text, so it doesn't appear again for the same user.
    CSS Styling: Use appropriate CSS to visually highlight the sentences in a user-friendly way, similar to Grammarly's highlighting system.

Below is the code for these functionalities:
1. Content Script: content.js

This script detects the sentences in the email body and highlights them, while also providing an option to dismiss them.

// content.js
document.addEventListener('DOMContentLoaded', function () {
  highlightSentences();
});

function highlightSentences() {
  // Select the body of the email (ensure you're targeting the right DOM element)
  const emailBody = document.querySelector('div[aria-label="Message Body"]');
  if (emailBody) {
    // Get all sentences or text fragments to highlight (you can improve this logic)
    const sentences = emailBody.textContent.split('.'); // Split text by period (.) to detect sentences

    sentences.forEach((sentence, index) => {
      if (sentence.trim()) {
        const span = document.createElement('span');
        span.textContent = sentence + '.'; // Append period back
        span.className = 'highlight-sentence'; // Add class for styling
        span.dataset.index = index; // Store sentence index to identify it

        // Append the sentence back to the email body, with a dismiss button
        const textNode = document.createTextNode(sentence);
        const dismissButton = createDismissButton(index);

        const wrapper = document.createElement('div');
        wrapper.className = 'sentence-wrapper';
        wrapper.appendChild(span);
        wrapper.appendChild(dismissButton);

        emailBody.appendChild(wrapper);
      }
    });
  }
}

// Create the dismiss button
function createDismissButton(index) {
  const button = document.createElement('button');
  button.textContent = 'Dismiss';
  button.className = 'dismiss-button';
  button.addEventListener('click', function () {
    dismissSentence(index);
  });
  return button;
}

// Function to remove the highlighted sentence and dismiss it
function dismissSentence(index) {
  const highlightedSentence = document.querySelector(`span[data-index="${index}"]`);
  const wrapper = highlightedSentence.closest('.sentence-wrapper');
  wrapper.style.display = 'none'; // Hide the dismissed sentence

  // Save the dismissed sentence in local storage (to prevent showing it again)
  let dismissedSentences = JSON.parse(localStorage.getItem('dismissedSentences')) || [];
  dismissedSentences.push(index);
  localStorage.setItem('dismissedSentences', JSON.stringify(dismissedSentences));
}

// Function to handle dismissed sentences (prevents them from showing up again)
function handleDismissedSentences() {
  const dismissedSentences = JSON.parse(localStorage.getItem('dismissedSentences')) || [];
  const allHighlightedSentences = document.querySelectorAll('.highlight-sentence');

  allHighlightedSentences.forEach((sentence, index) => {
    if (dismissedSentences.includes(index)) {
      sentence.closest('.sentence-wrapper').style.display = 'none'; // Hide the dismissed sentence
    }
  });
}

// Call the function when the content is loaded
handleDismissedSentences();

2. Background Script: background.js

Ensure the extension listens for interactions with the content page, handles any communication, and supports extension-wide logic.

chrome.runtime.onInstalled.addListener(function() {
  console.log("Safe Kids AI for Email Extension Installed");
});

// You can add additional background tasks if needed

3. Manifest File (manifest.json)

Define the permissions, content scripts, and other configurations in the manifest.

{
  "manifest_version": 3,
  "name": "Safe Kids AI for Email",
  "description": "Highlights sentences and provides dismiss functionality like Grammarly.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage"
  ],
  "content_scripts": [
    {
      "matches": ["https://mail.google.com/*"],
      "js": ["content.js"]
    }
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/16.png",
      "48": "icons/48.png",
      "128": "icons/128.png"
    }
  },
  "icons": {
    "16": "icons/16.png",
    "48": "icons/48.png",
    "128": "icons/128.png"
  }
}

4. CSS Styles (styles.css)

To make the highlighted text look similar to Grammarly’s highlighting style and provide a dismiss button.

/* styles.css */
.highlight-sentence {
  background-color: rgba(255, 223, 186, 0.7); /* Light yellow background */
  color: #333;
  border-radius: 3px;
  padding: 2px 5px;
}

.sentence-wrapper {
  display: flex;
  align-items: center;
  margin: 5px 0;
}

.dismiss-button {
  margin-left: 10px;
  background-color: #ff4d4d;
  color: white;
  border: none;
  padding: 2px 5px;
  border-radius: 3px;
  cursor: pointer;
}

.dismiss-button:hover {
  background-color: #ff3333;
}

5. Popup HTML (popup.html)

You can include an option to configure or view the extension's settings.

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Safe Kids AI for Email</title>
    <style>
      /* Popup styles */
    </style>
  </head>
  <body>
    <h3>Safe Kids AI for Email</h3>
    <p>Highlights sentences and allows for dismissing them for better focus.</p>
    <!-- Add any additional UI components here -->
  </body>
</html>

6. Storing and Managing Dismissed Sentences

This is done in the content script, using the localStorage to store the indices of dismissed sentences. This way, even if the page reloads, the dismissed sentences won’t be shown again.

// Save the dismissed sentence index to localStorage
let dismissedSentences = JSON.parse(localStorage.getItem('dismissedSentences')) || [];
dismissedSentences.push(index);
localStorage.setItem('dismissedSentences', JSON.stringify(dismissedSentences));

// Prevent dismissed sentences from appearing again
const dismissedSentences = JSON.parse(localStorage.getItem('dismissedSentences')) || [];

Steps to Install and Test the Chrome Extension

    Prepare the Extension Files:
        Ensure that all the code (manifest, content script, background script, CSS, etc.) is in a directory.
        Add all your assets (icons, etc.) to the correct directories.

    Load the Extension in Chrome:
        Open chrome://extensions/ in your browser.
        Enable "Developer mode".
        Click on "Load unpacked" and select the directory where you saved the extension files.

    Test the Extension:
        Go to Gmail and open an email.
        The extension should highlight sentences within the email.
        You can click the "Dismiss" button to hide the highlighted sentence. Dismissed sentences will be hidden the next time you visit the page.

Conclusion

With these updates, your Chrome extension should now be able to highlight text (similar to Grammarly) and allow users to dismiss it. The dismissed sentences will not appear again using local storage, which keeps track of the dismissed sentences.
