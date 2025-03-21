## Tickets

- [JIRA Ticket](URL)  
  *Contains the link to the related task or ticket for this Pull Request from the task management system (e.g., JIRA) to facilitate tracking.*

## Description

*Briefly describe the purpose of this Pull Request. Provide context about the changes made, explaining the reasons and objectives.*

**Example:**  
This PR adds the  Django library to ingest a datatable for displaying the lab detail list.

## Changes Made

*List the changes you have made in the code. Use a checklist to highlight the most important parts of the code and functionalities added or modified.*

**Example:**
- [x] Created a class  in  to handle the ingestion of the lab detail list.
- [x] Added a new URL in  for the .
- [x] Developed the  script to launch the new datatable in a modal when clicking an instance in the student roster datatable, handling data ingestion via AJAX.

## How to Test

*Provide clear, step-by-step instructions on how to test your changes. Include detailed steps so that reviewers or testers can replicate the functionality and verify the expected results.*

**Example:**
1. Navigate to the **Instructor's tab** from a user account with an active lab.
2. Verify that the table correctly displays the lab detail list.
3. Test the **search bar** functionality to ensure it filters the table data as expected.
4. Confirm that clicking on any student in the roster opens a modal with the correct lab details.

## Screenshots (optional)

*If applicable, include screenshots to visually demonstrate the changes you’ve made, especially if they affect the UI or include animated .GIFs showing how to test the functionality.*

**Example:**  
![Example Screenshot](https://example.com/screenshot.png)

## Dependencies (optional)

*List any dependencies this Pull Request relies on (e.g., libraries, packages, or other PRs). Indicate if any dependencies need to be added to configuration files like .*

**Example:**
- Ensure the  library is included in the requirements file or installed correctly.

## Additional Context (optional)

*Include any extra information that might be helpful for reviewers or testers (e.g., details about the test environment, known issues, or limitations).*

**Example:**  
Any additional notes or context that may help reviewers or testers.
