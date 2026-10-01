# Lab 11: Build and deploy a project status assistant with the Microsoft Teams SDK

**Estimated time:** 45–50 minutes

## Lab scenario

**Zava Retail** is a retail company with stores and digital commerce operations, supported by technology teams that manage multiple business and customer-facing projects. **Maya Kapoor, a project manager at Zava Retail,** manages several projects and needs to stay informed about project progress, risks, priorities, upcoming meetings, and stakeholder updates.

To help project teams work more efficiently, **Christine Parker, CTO** at Zava Retail, has asked the technology team to introduce a project-management assistant directly in Microsoft Teams, where teams already collaborate, hold meetings, and share project updates. Using the **Microsoft Teams SDK**, you will build a **Zava Project Status Assistant** that can support multiple projects and help users quickly access project information and take action from within Teams. For this lab, **Project Phoenix** is used as an example project with an **At Risk** status, a customer review scheduled for **tomorrow**, **three open issues**, and **two remaining deployment tasks**. You will build the assistant using Python, run it locally, expose it through a **Microsoft Dev Tunnel**, register it using the **Teams Developer CLI**, and interact with it directly in Microsoft Teams.

### Key persona

**Maya Kapoor — Project manager, Zava Retail**

Manages multiple projects and uses the **Zava Project Status Assistant** to quickly check project status, priorities, meeting preparation, and project updates.

## Objectives

By the end of this lab, you will be able to:

- Create a **Python Teams SDK** application.

- Build a **project-focused assistant** that can work with multiple Zava Retail projects.

- Run the assistant locally and expose it through a **Microsoft Dev Tunnel**.

- Register and deploy the application to **Microsoft Teams** using the Teams Developer CLI.

- Retrieve project status, priorities, and meeting preparation information directly in Teams.

- Generate and interact with a **project update using an Adaptive Card**.

## Exercise 1: Create the Teams SDK project

In this exercise, you create the foundation for the **Zava Project Status Assistant** using the **Microsoft Teams SDK for Python**. You also create a Python virtual environment and install the dependencies required to run the application.

### What is the Teams SDK?

Teams SDK is a suite of packages for building agents and applications on Microsoft Teams. It handles authentication, event routing, and Teams-specific plumbing so you can focus on your app's logic. Using it, you can create AI-powered agents, message extensions, embedded web apps, Adaptive Cards, dialogs, Microsoft Graph integrations, and more across TypeScript, C#, and Python.

### Task 1: Create a project

1.  Open Command Prompt and sign in to Teams using the following command:

```cmd
teams login
```

If the above command returns an error, use the following command instead:

```cmd
teams login --device-code
```

![Command Prompt running teams login --device-code](media/lab-11-exercise-01-task-01-step-01-teams-login.png)

2.  Open the link in your web browser, and then enter the code. Select the current username to sign in.

![Command Prompt showing the device login link and code, marked 1 and 2](media/lab-11-exercise-01-task-01-step-02-device-code-link.png)

![Enter code to allow access page with the code entered and the Next button highlighted](media/lab-11-exercise-01-task-01-step-02-enter-code.png)

![Pick an account page with the current user account highlighted](media/lab-11-exercise-01-task-01-step-02-pick-account.png)

3.  Select **Continue**, and then close the window.

![Are you trying to sign in to Teams-Toolkit prompt with the Continue button highlighted](media/lab-11-exercise-01-task-01-step-03-continue.png)

![Teams-Toolkit confirmation that you have signed in and may close the window](media/lab-11-exercise-01-task-01-step-03-signed-in.png)

4.  Run the following command to sign in to Azure:

```cmd
az login
```

5.  Check the status of the Teams sign-in. Make sure it shows **Sideloading: enabled**.

```cmd
teams status
```

![Command Prompt output of teams status showing the signed-in user and Sideloading enabled](media/lab-11-exercise-01-task-01-step-05-teams-status.png)

6.  After successfully signing in, create a new project using the following command:

```cmd
teams project new python project-assistant --template echo
```

7.  Enter **Y** and press **Enter**.

This confirms that you want to scaffold a new Python-based Teams app named `project-assistant` using the `echo` template.

8.  Move into the project folder:

```cmd
cd project-assistant
```

### Task 2: Install dependencies

1.  Create and activate a virtual environment:

```cmd
python -m venv .venv
.venv\Scripts\activate
```

![Command Prompt showing the project scaffolded and the virtual environment created and activated](media/lab-11-exercise-01-task-02-step-01-virtual-environment.png)

2.  Install all the dependencies using the following command:

```cmd
pip install -e .
```

![Command Prompt output of pip install -e . installing the project dependencies](media/lab-11-exercise-01-task-02-step-02-install-dependencies.png)

## Exercise 2: Build the project assistant

In this exercise, you will transform the starter Teams SDK echo application into a business-focused project assistant for Project Phoenix. You will add project data and message-handling logic to provide status updates, priorities, and meeting preparation.

### Task 1: Create the project assistant agent in Teams SDK

1.  Open Visual Studio Code and open the `project-assistant` folder from `C:\Users\demouser`.

![Visual Studio Code Open Folder dialog with the project-assistant folder and the Select Folder button highlighted](media/lab-11-exercise-02-task-01-step-01-open-folder.png)

2.  Navigate to `src\main.py` and replace the current content with the following code:

```python
from microsoft_teams.apps import App, ActivityContext
from microsoft_teams.api import MessageActivity

# ---------------------------------------------------------
# Create Teams application
# ---------------------------------------------------------
app = App(skip_auth=True)


# ---------------------------------------------------------
# Project Phoenix sample data
# ---------------------------------------------------------
PROJECT = {
    "name": "Project Phoenix",
    "owner": "Priya Nair",
    "status": "At Risk",
    "customer_review": "Tomorrow",
    "open_issues": 3,
    "deployment_tasks": 2,
    "next_milestone": "Production deployment",
}

# ---------------------------------------------------------
# Project status
# ---------------------------------------------------------
def project_status():
    return f"""
**Project Phoenix — Current Status**

**Overall status:** {PROJECT["status"]}

**Project owner:** {PROJECT["owner"]}

**Customer review:** {PROJECT["customer_review"]}

**Open issues:** {PROJECT["open_issues"]}

**Deployment tasks remaining:** {PROJECT["deployment_tasks"]}

**Next milestone:** {PROJECT["next_milestone"]}

**Recommended priority:**
Complete the remaining deployment tasks before the customer review.
"""

# ---------------------------------------------------------
# Meeting preparation
# ---------------------------------------------------------
def meeting_preparation():
    return """
**Phoenix Project Meeting Preparation**

**Meeting purpose**

Review customer-release readiness for Project Phoenix.

**Key discussion topics**

• Deployment readiness
• Open issues
• Customer review preparation
• Production deployment

**Open decisions**

• Confirm owners for the remaining deployment tasks.
• Confirm that critical issues are resolved.
• Confirm readiness of the customer review package.

**Questions to raise**

1. Are all remaining deployment tasks assigned?
2. Are the three open issues being actively tracked?
3. Is the customer review package ready?
4. Are there any risks that could affect production deployment?
"""

# ---------------------------------------------------------
# Today's priorities
# ---------------------------------------------------------
def priorities():
    return """
**Today's Phoenix Priorities**

1. Complete the remaining deployment tasks.
2. Review the three open issues.
3. Confirm customer-review readiness.
4. Verify the production deployment checklist.

**Why this matters**

The customer review is tomorrow, so deployment readiness is currently
the highest priority.
"""


# ---------------------------------------------------------
# Normal Teams message handler
# ---------------------------------------------------------
@app.on_message
async def handle_message(ctx: ActivityContext[MessageActivity]):
    text = (ctx.activity.text or "").lower().strip()

    print(f"[USER] {ctx.activity.text}")

    if (
        "status" in text
        or "project status" in text
        or "how is phoenix" in text
    ):
        await ctx.send(project_status())
        return

    if (
        "meeting" in text
        or "prepare me" in text
        or "meeting preparation" in text
    ):
        await ctx.send(meeting_preparation())
        return

    if (
        "priority" in text
        or "priorities" in text
        or "what should i focus" in text
        or "what should i do" in text
    ):
        await ctx.send(priorities())
        return

    await ctx.send(
        "I can help with Project Phoenix.\n\n"
        "Try:\n"
        "• What is the current status of Project Phoenix?\n"
        "• Prepare me for my Phoenix project meeting.\n"
        "• What should I focus on today?"
    )

# ---------------------------------------------------------
# Start application
# ---------------------------------------------------------
if __name__ == "__main__":
    import asyncio

    asyncio.run(app.start())
```

> **NOTE**
>
> `skip_auth=True` is intended for local unauthenticated Playground testing, not the Teams channel. When you move the agent to production, remove it.

![main.py open in Visual Studio Code with the new project assistant code](media/lab-11-exercise-02-task-01-step-02-main-py.png)

## Exercise 3: Understand the project assistant code (read only)

Now that you have created the project assistant, let's review its structure and key components. Understanding how these components work together will help you extend the assistant with additional capabilities.

### Project structure

The project assistant uses a simple project structure:

```text
project-assistant/
└── src/
    └── main.py    # Main application code
```

- `src/`: Contains the application source code.

- `main.py`: The entry point of the application. It defines the project information, response functions, message handling, and application startup.

### The App class

The heart of an application is the `App` class. This class handles all incoming activities and manages the application's lifecycle. It also acts as a way to host your application service.

```python
from microsoft_teams.apps import App, ActivityContext
from microsoft_teams.api import MessageActivity

app = App()
```

The app configuration includes a variety of options that allow you to customize its behavior, including controlling the underlying server, authentication, and other settings.

![main.py with the import statements and the App creation highlighted](media/lab-11-exercise-03-app-class.png)

### Project information

The assistant needs information about Project Phoenix to provide useful responses. This information is stored in a Python dictionary.

```python
PROJECT = {
    "name": "Project Phoenix",
    "owner": "Priya Nair",
    "status": "At Risk",
    "customer_review": "Tomorrow",
    "open_issues": 3,
    "deployment_tasks": 2,
    "next_milestone": "Production deployment",
}
```

Each key represents a piece of project information that the assistant can use when generating responses. This approach also makes it easy to update the project details without changing the message-handling logic.

![main.py with the project_status function highlighted](media/lab-11-exercise-03-response-functions.png)

### Response functions

The assistant uses separate Python functions to generate responses for different types of requests. For example:

```python
def project_status():
    return f"""
**Project Phoenix — Current Status**

**Overall status:** {PROJECT["status"]}

**Project owner:** {PROJECT["owner"]}

**Customer review:** {PROJECT["customer_review"]}
"""
```

The application also includes functions for meeting preparation and daily priorities:

- `project_status()`: Provides the current project status.

- `meeting_preparation()`: Provides information to help prepare for a project meeting.

- `priorities()`: Provides the recommended project priorities.

Separating these functions keeps the response logic organized and makes the application easier to extend.

![main.py with the project_status function highlighted](media/lab-11-exercise-03-response-functions.png)

### Message handling

Teams applications can respond to different types of activities. In this application, the assistant responds to incoming messages using the `@app.on_message` handler.

```python
@app.on_message
async def handle_message(ctx: ActivityContext[MessageActivity]):
    text = (ctx.activity.text or "").lower().strip()
```

The handler receives the incoming message through the `ActivityContext` and reads the text sent by the user. The application then checks the message to determine what information the user is requesting. For example:

```python
if "status" in text:
    await ctx.send(project_status())
    return
```

When a user asks about the project status, the application calls `project_status()` and sends the result back to the user. The same approach is used to handle meeting preparation and priority-related questions.

![main.py with the @app.on_message handler and its keyword conditions highlighted](media/lab-11-exercise-03-message-handling.png)

### Sending responses

The `ctx.send()` method sends a message from the assistant back to the user in Teams.

```python
await ctx.send(project_status())
```

In this example, the `project_status()` function generates the response and `ctx.send()` delivers it to the Teams conversation.

### Fallback response

Not every user message will match the conditions defined in the application. When no matching keyword is found, the assistant provides examples of questions that it can handle.

```python
await ctx.send(
    "I can help with Project Phoenix.\n\n"
    "Try:\n"
    "• What is the current status of Project Phoenix?\n"
    "• Prepare me for my Phoenix project meeting.\n"
    "• What should I focus on today?"
)
```

This gives the user guidance instead of leaving the message unanswered.

![main.py with the fallback ctx.send response highlighted](media/lab-11-exercise-03-fallback-response.png)

### Application lifecycle

The application starts when `main.py` is executed.

```python
if __name__ == "__main__":
    import asyncio

    asyncio.run(app.start())
```

This code initializes your application server and, when configured for Teams, also authenticates it so it is ready to send and receive messages.

![main.py with the application start block highlighted](media/lab-11-exercise-03-application-lifecycle.png)

## Exercise 4: Run the agent locally

In this exercise, you start the Python application and verify its message-handling logic before connecting it to Microsoft Teams. Testing locally first helps you catch code or dependency issues before you introduce the Dev Tunnel and Teams registration steps.

1.  Go back to Command Prompt and start the development server using the following command:

```cmd
python .\src\main.py
```

> **NOTE**
>
> Keep this window running; do not close it.

![Command Prompt running python .\src\main.py with the server started](media/lab-11-exercise-04-step-01-start-server.png)

2.  In the console, you should see output similar to the following:

```text
The HTTP server is now listening on port 3978.
```

3.  Open a new Command Prompt window. You will test your agent locally, without sideloading it into Teams, using the Microsoft 365 Agents Playground. Run the following command to open the Microsoft 365 Agents Playground:

```cmd
agentsplayground -e http://localhost:3978/api/messages -c emulator
```

![Command Prompt running the agentsplayground command and launching the playground](media/lab-11-exercise-04-step-03-agents-playground-command.png)

4.  Enter the following prompts:

Prompt 1: +++Hello+++

![Microsoft 365 Agents Playground with Hello typed in the message box](media/lab-11-exercise-04-step-04-hello-prompt.png)

![Microsoft 365 Agents Playground showing the assistant fallback response to Hello](media/lab-11-exercise-04-step-04-hello-response.png)

Prompt 2: +++What is the current status of Project Phoenix?+++

![Microsoft 365 Agents Playground with the Project Phoenix status question typed in the message box](media/lab-11-exercise-04-step-04-status-prompt.png)

![Microsoft 365 Agents Playground showing the Project Phoenix current status response](media/lab-11-exercise-04-step-04-status-response.png)

5.  Close the window to end the Microsoft 365 Agents Playground, and press **CTRL+C** in both Command Prompt windows to stop the current processes.

## Exercise 5: Add the Adaptive Card

In this exercise, you extend the project assistant with an interactive Adaptive Card. The card presents a project update in a structured format and lets the user choose whether to send or cancel the update. This introduces a practical Teams interaction beyond plain text messages.

### Task 1: Add the Adaptive Card to the agent

1.  Open Visual Studio Code again and open the `main.py` file.

2.  Replace the existing import section at the very top of `main.py` with the following code:

```python
from microsoft_teams.apps import App, ActivityContext
from microsoft_teams.api import (
    MessageActivity,
    AdaptiveCardInvokeActivity,
    AdaptiveCardActionMessageResponse,
    InvokeResponse,
)
from microsoft_teams.cards import AdaptiveCard, TextBlock, ExecuteAction
```

This adds the Teams SDK classes required to create an Adaptive Card and handle button actions.

![main.py with the new import section highlighted](media/lab-11-exercise-05-task-01-step-02-imports.png)

3.  Add this entire block after the `priorities()` function and before the `@app.on_message` handler:

```python
# ---------------------------------------------------------
# Adaptive Card
# ---------------------------------------------------------

def project_update_card():

    return AdaptiveCard(
        version="1.5",

        body=[
            TextBlock(
                text="Project Phoenix — Team Update",
                weight="Bolder",
                size="Large",
                wrap=True,
            ),

            TextBlock(
                text="Status: AT RISK",
                weight="Bolder",
                wrap=True,
            ),

            TextBlock(
                text=(
                    "Customer review: Tomorrow\n"
                    "Open issues: 3\n"
                    "Deployment tasks remaining: 2\n"
                    "Next milestone: Production deployment"
                ),
                wrap=True,
            ),

            TextBlock(
                text=(
                    "Priority: Complete the remaining deployment "
                    "tasks before the customer review."
                ),
                wrap=True,
            ),
        ],

        actions=[
            ExecuteAction(
                title="Send Update",
                verb="send_update",
                data={
                    "action": "send_update"
                },
            ),

            ExecuteAction(
                title="Cancel",
                verb="cancel_update",
                data={
                    "action": "cancel_update"
                },
            ),
        ],
    )
```

This creates an Adaptive Card containing the current Phoenix project status and two interactive buttons: **Send Update** and **Cancel**.

![main.py with the project_update_card function highlighted](media/lab-11-exercise-05-task-01-step-03-card-function.png)

4.  Inside `handle_message()`, add the following code after the existing priorities condition and before the default response:

```python
    # Adaptive Card
    if (
        "project update" in text
        or "draft an update" in text
        or "draft a project update" in text
    ):
        await ctx.send(project_update_card())
        return
```

This connects a message such as **Draft a project update** to the Adaptive Card.

![main.py with the Adaptive Card condition inside handle_message highlighted](media/lab-11-exercise-05-task-01-step-04-card-condition.png)

5.  Add the following Adaptive Card action handler before the `Start application` section (`if __name__ == "__main__":`):

```python
# ---------------------------------------------------------
# Adaptive Card action handler
# ---------------------------------------------------------

@app.on_card_action
async def handle_card_action(
    ctx: ActivityContext[AdaptiveCardInvokeActivity],
) -> InvokeResponse[AdaptiveCardActionMessageResponse]:

    action = ctx.activity.value.action

    data = action.data or {}

    action_name = data.get("action")

    print(f"[CARD ACTION] {action_name}")

    # -----------------------------------------------------
    # Send Update
    # -----------------------------------------------------

    if action_name == "send_update":

        return InvokeResponse(
            status=200,
            body=AdaptiveCardActionMessageResponse(
                value=(
                    "✅ **Project update sent successfully.**\n\n"
                    "The Project Phoenix team has been notified "
                    "of the current project status."
                )
            ),
        )

    # -----------------------------------------------------
    # Cancel
    # -----------------------------------------------------

    if action_name == "cancel_update":

        return InvokeResponse(
            status=200,
            body=AdaptiveCardActionMessageResponse(
                value=(
                    "❌ **Project update cancelled.**\n\n"
                    "No update was sent to the Project Phoenix team."
                )
            ),
        )

    # -----------------------------------------------------
    # Unknown action
    # -----------------------------------------------------

    return InvokeResponse(
        status=200,
        body=AdaptiveCardActionMessageResponse(
            value="⚠️ Unknown card action."
        ),
    )
```

This handler processes the button selection and returns an appropriate response when the user chooses **Send Update** or **Cancel**.

![main.py with the @app.on_card_action handler highlighted](media/lab-11-exercise-05-task-01-step-05-card-action-handler.png)

6.  Save the file using **CTRL+S**.

7.  The complete `main.py` file now looks like this:

```python
from microsoft_teams.apps import App, ActivityContext
from microsoft_teams.api import (
    MessageActivity,
    AdaptiveCardInvokeActivity,
    AdaptiveCardActionMessageResponse,
    InvokeResponse,
)
from microsoft_teams.cards import AdaptiveCard, TextBlock, ExecuteAction

# ---------------------------------------------------------
# Create Teams application
# ---------------------------------------------------------
app = App(skip_auth=True)


# ---------------------------------------------------------
# Project Phoenix sample data
# ---------------------------------------------------------
PROJECT = {
    "name": "Project Phoenix",
    "owner": "Priya Nair",
    "status": "At Risk",
    "customer_review": "Tomorrow",
    "open_issues": 3,
    "deployment_tasks": 2,
    "next_milestone": "Production deployment",
}

# ---------------------------------------------------------
# Project status
# ---------------------------------------------------------
def project_status():
    return f"""
**Project Phoenix — Current Status**

**Overall status:** {PROJECT["status"]}

**Project owner:** {PROJECT["owner"]}

**Customer review:** {PROJECT["customer_review"]}

**Open issues:** {PROJECT["open_issues"]}

**Deployment tasks remaining:** {PROJECT["deployment_tasks"]}

**Next milestone:** {PROJECT["next_milestone"]}

**Recommended priority:**
Complete the remaining deployment tasks before the customer review.
"""

# ---------------------------------------------------------
# Meeting preparation
# ---------------------------------------------------------
def meeting_preparation():
    return """
**Phoenix Project Meeting Preparation**

**Meeting purpose**

Review customer-release readiness for Project Phoenix.

**Key discussion topics**

• Deployment readiness
• Open issues
• Customer review preparation
• Production deployment

**Open decisions**

• Confirm owners for the remaining deployment tasks.
• Confirm that critical issues are resolved.
• Confirm readiness of the customer review package.

**Questions to raise**

1. Are all remaining deployment tasks assigned?
2. Are the three open issues being actively tracked?
3. Is the customer review package ready?
4. Are there any risks that could affect production deployment?
"""

# ---------------------------------------------------------
# Today's priorities
# ---------------------------------------------------------
def priorities():
    return """
**Today's Phoenix Priorities**

1. Complete the remaining deployment tasks.
2. Review the three open issues.
3. Confirm customer-review readiness.
4. Verify the production deployment checklist.

**Why this matters**

The customer review is tomorrow, so deployment readiness is currently
the highest priority.
"""


# ---------------------------------------------------------
# Adaptive Card
# ---------------------------------------------------------

def project_update_card():

    return AdaptiveCard(
        version="1.5",

        body=[
            TextBlock(
                text="Project Phoenix — Team Update",
                weight="Bolder",
                size="Large",
                wrap=True,
            ),

            TextBlock(
                text="Status: AT RISK",
                weight="Bolder",
                wrap=True,
            ),

            TextBlock(
                text=(
                    "Customer review: Tomorrow\n"
                    "Open issues: 3\n"
                    "Deployment tasks remaining: 2\n"
                    "Next milestone: Production deployment"
                ),
                wrap=True,
            ),

            TextBlock(
                text=(
                    "Priority: Complete the remaining deployment "
                    "tasks before the customer review."
                ),
                wrap=True,
            ),
        ],

        actions=[
            ExecuteAction(
                title="Send Update",
                verb="send_update",
                data={
                    "action": "send_update"
                },
            ),

            ExecuteAction(
                title="Cancel",
                verb="cancel_update",
                data={
                    "action": "cancel_update"
                },
            ),
        ],
    )


# ---------------------------------------------------------
# Normal Teams message handler
# ---------------------------------------------------------
@app.on_message
async def handle_message(
    ctx: ActivityContext[MessageActivity]
):
    text = (ctx.activity.text or "").lower().strip()

    print(f"[USER] {ctx.activity.text}")

    # Project status
    if (
        "status" in text
        or "project status" in text
        or "how is phoenix" in text
    ):
        await ctx.send(project_status())
        return

    # Meeting preparation
    if (
        "meeting" in text
        or "prepare me" in text
        or "meeting preparation" in text
    ):
        await ctx.send(meeting_preparation())
        return

    # Priorities
    if (
        "priority" in text
        or "priorities" in text
        or "what should i focus" in text
        or "what should i do" in text
    ):
        await ctx.send(priorities())
        return

    # Adaptive Card
    if (
        "project update" in text
        or "draft an update" in text
        or "draft a project update" in text
    ):
        await ctx.send(project_update_card())
        return

    # Default response
    await ctx.send(
        "I can help with Project Phoenix.\n\n"
        "Try:\n"
        "• What is the current status of Project Phoenix?\n"
        "• Prepare me for my Phoenix project meeting.\n"
        "• What should I focus on today?\n"
        "• Draft a project update."
    )


# ---------------------------------------------------------
# Adaptive Card action handler
# ---------------------------------------------------------

@app.on_card_action
async def handle_card_action(
    ctx: ActivityContext[AdaptiveCardInvokeActivity],
) -> InvokeResponse[AdaptiveCardActionMessageResponse]:

    action = ctx.activity.value.action

    data = action.data or {}

    action_name = data.get("action")

    print(f"[CARD ACTION] {action_name}")

    # -----------------------------------------------------
    # Send Update
    # -----------------------------------------------------

    if action_name == "send_update":

        return InvokeResponse(
            status=200,
            body=AdaptiveCardActionMessageResponse(
                value=(
                    "✅ **Project update sent successfully.**\n\n"
                    "The Project Phoenix team has been notified "
                    "of the current project status."
                )
            ),
        )

    # -----------------------------------------------------
    # Cancel
    # -----------------------------------------------------

    if action_name == "cancel_update":

        return InvokeResponse(
            status=200,
            body=AdaptiveCardActionMessageResponse(
                value=(
                    "❌ **Project update cancelled.**\n\n"
                    "No update was sent to the Project Phoenix team."
                )
            ),
        )

    # -----------------------------------------------------
    # Unknown action
    # -----------------------------------------------------

    return InvokeResponse(
        status=200,
        body=AdaptiveCardActionMessageResponse(
            value="⚠️ Unknown card action."
        ),
    )


# ---------------------------------------------------------
# Start application
# ---------------------------------------------------------
if __name__ == "__main__":
    import asyncio

    asyncio.run(app.start())
```

### Task 2: Change the application from local Playground mode to Teams mode

1.  Open the `main.py` file and look for the following line: `app = App(skip_auth=True)`.

2.  Remove `skip_auth=True`, because it is only used for unauthenticated local Playground testing, not for a real Teams deployment. Your `App()` line should look like this:

```python
app = App()
```

This matters because the Dev Tunnel is public. A public tunnel exposes the local port, so authentication should remain enabled for real channel testing.

![main.py with app = App() highlighted after removing skip_auth](media/lab-11-exercise-05-task-02-step-02-app-without-skip-auth.png)

## Exercise 6: Deploy the agent to Teams

In this exercise, you make the assistant reachable from Microsoft Teams. You will expose the local application through a Microsoft Dev Tunnel, register the bot infrastructure with the Teams Developer CLI, and install the app in Teams for end-to-end testing.

### Task 1: Create the Microsoft Dev Tunnel

1.  Open a new Command Prompt window and navigate to the project folder:

```cmd
cd C:\Users\demouser\project-assistant
```

![Command Prompt after changing to the project-assistant folder](media/lab-11-exercise-06-task-01-step-01-project-folder.png)

2.  Sign in to the dev tunnel service using the following command:

```cmd
devtunnel user login
```

![Command Prompt running devtunnel user login](media/lab-11-exercise-06-task-01-step-02-devtunnel-login.png)

3.  Select the current username and select **Continue**.

![Let's get you signed in dialog with the current account and the Continue button highlighted](media/lab-11-exercise-06-task-01-step-03-select-account.png)

![Command Prompt confirming that you are logged in to the dev tunnel service](media/lab-11-exercise-06-task-01-step-03-logged-in.png)

4.  Start the dev tunnel using the following command:

```cmd
devtunnel host -p 3978 --allow-anonymous
```

> **NOTE**
>
> Do not close this window; keep it running.

![Command Prompt running devtunnel host with the tunnel ready to accept connections](media/lab-11-exercise-06-task-01-step-04-devtunnel-host.png)

5.  Save the dev tunnel URL in Notepad; you will use it in the next task.

![Command Prompt with the dev tunnel connect URL highlighted](media/lab-11-exercise-06-task-01-step-05-tunnel-url.png)

### Task 2: Restart the Python agent

1.  Go back to the Command Prompt window where you ran the development server, and run the following command again:

```cmd
python .\src\main.py
```

Keep this window running.

![Command Prompt running python .\src\main.py again with the server started](media/lab-11-exercise-06-task-02-step-01-restart-agent.png)

### Task 3: Register and install the Teams application

1.  Open a new Command Prompt window and navigate to the project folder:

```cmd
cd C:\Users\demouser\project-assistant
```

![Command Prompt after changing to the project-assistant folder](media/lab-11-exercise-06-task-03-step-01-project-folder.png)

2.  Make sure your Python environment is activated:

```cmd
.venv\Scripts\activate
```

![Command Prompt with the virtual environment activated](media/lab-11-exercise-06-task-03-step-02-activate-environment.png)

3.  Register the app using the following command. Replace `<tunnel-host>` with **your actual tunnel host**:

```cmd
teams app create --name Project-Assistant --endpoint https://<tunnel-host>/api/messages --env .env
```

For example:

```cmd
teams app create --name Project-Assistant --endpoint https://c29q1k0t-3978.use2.devtunnels.ms/api/messages --env .env
```

Microsoft's current registration quickstart uses `teams app create` with the public tunnel endpoint and `.env` for Python projects. The command creates and registers the bot infrastructure and prints the Teams app ID plus an **Install in Teams** link.

![Command Prompt running teams app create with the tunnel endpoint](media/lab-11-exercise-06-task-03-step-03-teams-app-create.png)

4.  Enter **y** to confirm the new application.

![Confirm creation prompt with y entered](media/lab-11-exercise-06-task-03-step-04-confirm-creation.png)

5.  Select **Install in Teams** to install the app in Microsoft Teams.

> **NOTE**
>
> If you lose the **Install in Teams** link, use the following command:

```cmd
teams app get <YOUR-TEAMS-APP-ID> --install-link
```

![Command Prompt output with the app created, credentials written to .env, and the Install in Teams link highlighted](media/lab-11-exercise-06-task-03-step-05-install-in-teams.png)

6.  Select **Use the web app instead**.

![Microsoft Teams page with the Use the web app instead button highlighted](media/lab-11-exercise-06-task-03-step-06-use-web-app.png)

7.  Select **Add**.

![Project-Assistant app details with the Add button highlighted](media/lab-11-exercise-06-task-03-step-07-add-app.png)

8.  Select **Open**.

![Project-Assistant added successfully dialog with the Open button highlighted](media/lab-11-exercise-06-task-03-step-08-open-app.png)

### Task 4: Verify the generated .env file

1.  Go to Visual Studio Code and open the `.env` file to see the credentials generated by the Teams Developer CLI.

The Python application will use these credentials when authentication is enabled.

## Exercise 7: Validate the assistant end to end

Now that the app is installed in Teams, validate the complete flow from a Teams message to the Python application and back to the Teams client. Use the prompts below in order. After each prompt, confirm that the response matches the expected behavior.

1.  To check the project status, enter the following prompt:

+++What is the current status of Project Phoenix?+++

![Project-Assistant chat in Teams showing the fallback response to Hello](media/lab-11-exercise-07-step-01-status-prompt.png)

![Project-Assistant chat in Teams showing the Project Phoenix current status response](media/lab-11-exercise-07-step-01-status-response.png)

2.  To prepare for the project meeting, enter the following prompt:

+++Prepare me for my Phoenix project meeting.+++

![Project-Assistant chat in Teams showing the Phoenix project meeting preparation response](media/lab-11-exercise-07-step-02-meeting-preparation.png)

3.  To generate the project update card, enter the following prompt:

+++Draft a project update.+++

![Project-Assistant chat in Teams showing the Project Phoenix team update Adaptive Card](media/lab-11-exercise-07-step-03-project-update-card.png)

4.  Select the **Send Update** button to test the card actions. The assistant returns a confirmation that the project update was sent successfully.

![Project Phoenix team update Adaptive Card with the Send Update button highlighted](media/lab-11-exercise-07-step-04-send-update.png)

![Adaptive Card showing the confirmation that the project update was sent successfully](media/lab-11-exercise-07-step-04-update-sent.png)

## Summary

In this lab, you built and deployed a **Zava Project Status Assistant** using the **Microsoft Teams SDK for Python**. You created a Teams application that provides Project Phoenix status, priorities, and meeting-preparation information, and then enhanced it with an **Adaptive Card** that enables users to take action on a project update. You tested the application locally, exposed it through a **Microsoft Dev Tunnel**, and registered and installed it in **Microsoft Teams** using the Teams Developer CLI. Finally, you completed an end-to-end test in Teams to verify that the assistant can respond to project-related requests and process Adaptive Card actions.
