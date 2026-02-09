# Step 03: Your First AI-Assisted Coding Session

**Level:** Beginner
**Time:** ~25 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Set up Claude Code or Cursor for AI-assisted development
2. Write your first effective prompt for code generation
3. Review and understand AI-generated code before using it
4. Develop iteration and refinement habits

---

In [Step 02](02-how-llms-work-for-code.md), you learned how LLMs predict tokens and why we must "treat AI output as untrusted input." Now it's time to put that knowledge into practice with your first hands-on coding session.

## The Tool Landscape

Several tools enable AI-assisted coding. Here are the major options:

### Claude Code (Anthropic)

**What it is:** A command-line tool that gives Claude access to your terminal, files, and development environment.

**Best for:**
- Terminal-focused workflows
- Direct file system access
- Complex multi-file changes
- Developers comfortable with CLI

### Cursor

**What it is:** An AI-native IDE built on VS Code with deep AI integration.

**Best for:**
- Visual development workflows
- Inline code completion
- Developers who prefer IDEs
- Multi-model support (Claude, GPT-4, etc.)

### GitHub Copilot

**What it is:** AI pair programmer integrated into VS Code, JetBrains, and other editors.

**Best for:**
- Inline autocomplete
- Suggestion-based workflows
- Teams already on GitHub

### Windsurf (Codeium)

**What it is:** AI-powered IDE with autonomous agent capabilities.

**Best for:**
- Autonomous coding flows
- Large codebase navigation

### Which Should You Start With?

| Preference | Recommended Tool |
|------------|-----------------|
| Command line lover | Claude Code |
| VS Code user | Cursor or Copilot |
| Want maximum AI power | Claude Code or Cursor |
| Team standardization | Whatever your team uses |

### Tool Comparison Matrix

```
+-------------------------------------------------------------------------+
|                         AI CODING TOOL COMPARISON                       |
+-------------------------------------------------------------------------+
|                                                                         |
|                  CLI    IDE    Autocomplete   Agent Mode   Multi-Model  |
|                                                                         |
|  Claude Code     ██     --        --            ██           --         |
|                  Primary interface is terminal, strong agent mode       |
|                                                                         |
|  Cursor          --     ██        ██            ██           ██         |
|                  VS Code fork with deep AI integration                  |
|                                                                         |
|  Copilot         --     ██        ██            ░░           --         |
|                  Best-in-class autocomplete, agent emerging             |
|                                                                         |
|  Windsurf        --     ██        ██            ██           ██         |
|                  Strong autonomous capabilities                         |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  Legend:  ██ = Primary Strength   ░░ = Partial/Emerging   -- = Not Core |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  QUICK SELECTION GUIDE:                                                 |
|                                                                         |
|  "I love the terminal"              --> Claude Code                     |
|  "I want inline suggestions"        --> Cursor or Copilot               |
|  "I want AI to do multi-step tasks" --> Claude Code, Cursor, or Windsurf|
|  "I need multiple models"           --> Cursor or Windsurf              |
|  "My team already uses X"           --> Use X                           |
|                                                                         |
+-------------------------------------------------------------------------+
```

This guide focuses on **Claude Code** and **Cursor** as primary examples, but the concepts apply to all tools.

### Understanding Workbench Types: ACE vs. AEE

The SASE (Structured Agentic Software Engineering) framework [^sase] distinguishes two types of AI development environments, which helps explain why different tools feel different:

**ACE (AI-assisted Coding Environment)**
- Human remains the primary driver
- AI assists with suggestions, completions, and explanations
- Examples: Copilot in suggestion mode, basic IDE integrations
- Best for: Developers who want AI help but prefer to stay in control

**AEE (Agentic Engineering Environment)**
- AI can take multi-step autonomous actions
- Human provides goals; AI determines approach
- Examples: Claude Code, Cursor Agent mode, Windsurf Cascade
- Best for: Tasks that benefit from AI planning and executing sequences of actions

Most modern tools blur this line, offering both modes. Cursor can work as an ACE (inline completions) or AEE (agent mode). Claude Code is primarily an AEE but can answer simple questions like an ACE.

**As a beginner:** Start in ACE mode (simple prompts, review each suggestion). Gradually explore AEE features as you build trust and experience.

---

## Installing Claude Code

### Prerequisites

- macOS, Linux, or Windows (with WSL)
- Node.js 18 or later
- A terminal application

### Installation Steps

1. **Install via npm:**
```bash
npm install -g @anthropic-ai/claude-code
```

2. **Authenticate with Anthropic:**
```bash
claude login
```
Follow the prompts to sign in with your Anthropic account.

3. **Verify installation:**
```bash
claude --version
```

4. **Start Claude Code:**
```bash
cd /path/to/your/project
claude
```

### First Run

When you run `claude` for the first time:
- It will scan your project structure
- It will look for `CLAUDE.md` files (we'll create these in Step 05)
- You'll see a prompt where you can start typing

### Claude Code Troubleshooting

| Problem | Solution |
|---------|----------|
| "command not found: claude" | Verify npm global path is in your PATH. Run `npm config get prefix` and add `/bin` to your PATH |
| Login fails or times out | Check your internet connection; verify you have an Anthropic account at console.anthropic.com |
| "Permission denied" errors | On macOS/Linux, you may need to grant terminal access in Security settings |
| Slow responses | Check Anthropic status page; try `/clear` to reduce context size |
| "Context too large" errors | Use `/compact` to summarize conversation, or start fresh with `/clear` |
| API key issues | Run `claude logout` then `claude login` to re-authenticate |

### Useful Claude Code Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/clear` | Clear conversation history (start fresh) |
| `/compact` | Compress conversation while preserving key context |
| `/model` | Switch between available models |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit Claude Code |

### Terminal Workflow Example

Here's what a typical Claude Code session looks like. Understanding this workflow helps you work more effectively.

```bash
# Start a Claude Code session in your project directory
$ cd ~/projects/my-api
$ claude

╭─────────────────────────────────────────────────────────────╮
│ Claude Code v1.0.0                                          │
│ Working directory: ~/projects/my-api                        │
│ Type /help for commands, Ctrl+D to exit                    │
╰─────────────────────────────────────────────────────────────╯

> What files are in this project?

I'll examine the project structure.

Reading directory structure...

This project contains:
- src/
  - main.py
  - routes/
    - users.py
    - auth.py
  - models/
    - user.py
- tests/
  - test_users.py
- requirements.txt
- README.md

It appears to be a FastAPI application with user authentication.

> Show me the User model

Reading src/models/user.py...

```python
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
```

> Add a `last_login` field to the User model

I'll add the last_login field to the User model.

Editing src/models/user.py...

Added the following changes:
- Added `last_login` column (DateTime, nullable, no default)

```python
# Updated User model
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    last_login = Column(DateTime, nullable=True)  # New field
```

Do you want me to also update the auth route to set last_login on successful login?

> Yes, please update the auth route

Reading src/routes/auth.py...
Editing src/routes/auth.py...

Updated the login endpoint to set `last_login`:

```python
@router.post("/login")
async def login(credentials: LoginRequest, db: Session = Depends(get_db)):
    user = authenticate_user(db, credentials.email, credentials.password)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid credentials")

    # Update last login time
    user.last_login = datetime.utcnow()
    db.commit()

    token = create_access_token(user.id)
    return {"access_token": token, "token_type": "bearer"}
```

> Run the tests to make sure nothing broke

Running pytest tests/...

================================ test session ================================
tests/test_users.py::test_create_user PASSED
tests/test_users.py::test_get_user PASSED
tests/test_users.py::test_user_not_found PASSED

3 passed in 0.42s

All tests pass! Note that you should add tests for the new last_login
functionality. Would you like me to generate those tests?

> /compact

Compacting conversation...

Preserved context:
- Project: FastAPI API with user authentication
- Changes made: Added last_login field to User model and auth route
- Current state: Tests passing

Context size reduced from 12,847 tokens to 2,156 tokens.

> Create a test for the last_login update

Creating tests/test_auth.py...

```python
# tests/test_auth.py
import pytest
from datetime import datetime, timedelta
from src.models.user import User

def test_login_updates_last_login(client, test_user, db):
    """Successful login should update the user's last_login timestamp."""
    # Record time before login
    before_login = datetime.utcnow()

    # Perform login
    response = client.post("/auth/login", json={
        "email": test_user.email,
        "password": "testpassword"
    })
    assert response.status_code == 200

    # Verify last_login was updated
    db.refresh(test_user)
    assert test_user.last_login is not None
    assert test_user.last_login >= before_login
    assert test_user.last_login <= datetime.utcnow()
```

> /clear

Conversation cleared. Starting fresh.

>
```

**Key Terminal Workflow Patterns:**

| Pattern | When to Use |
|---------|------------|
| Start specific | Navigate to your project directory before running `claude` |
| Ask before acting | Claude explains what it will do before making changes |
| Incremental changes | Make small changes, verify, then continue |
| `/compact` | When context grows large but you want to continue the task |
| `/clear` | When switching to a completely different task |
| Run tests | Always verify changes don't break existing functionality |

---

## Installing Cursor

### Installation Steps

1. **Download Cursor:**
   - Go to [cursor.com](https://cursor.com)
   - Download the installer for your platform

2. **Install and launch:**
   - Run the installer
   - Open Cursor

3. **Configure AI model:**
   - Open Settings (Cmd/Ctrl + ,)
   - Navigate to "AI" settings
   - Add your API keys or use the built-in models

4. **Open your project:**
   - File → Open Folder → Select your project

### Cursor Troubleshooting

| Problem | Solution |
|---------|----------|
| Completions not appearing | Check that AI features are enabled in Settings > AI |
| API key errors | Verify key format; regenerate key if needed from your provider's console |
| "Model not available" | Some models require subscription; check your plan |
| Slow or unresponsive | Large files may overwhelm context; try selecting specific code ranges |
| Agent mode stuck | Press Escape to cancel; try a simpler, more focused prompt |
| Conflicting extensions | Disable other AI extensions (Copilot, etc.) that might interfere |

### Cursor Key Features

| Feature | Access | Use Case |
|---------|--------|----------|
| Inline completion | Type and wait | Quick suggestions while coding |
| Chat panel | Cmd/Ctrl + L | Ask questions, explain code |
| Composer | Cmd/Ctrl + I | Multi-file edits, larger changes |
| Agent mode | Enable in Composer | Autonomous multi-step tasks |
| @ references | @file, @folder, @web | Include specific context in prompts |

---

## Anatomy of a Prompt

A good prompt for code generation has three components:

### Prompt Anatomy Diagram

```text
+-------------------------------------------------------------------------+
|                         PROMPT ANATOMY                                  |
+-------------------------------------------------------------------------+
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |                         1. CONTEXT                                |  |
|  |                                                                   |  |
|  |  What the AI needs to know about your situation                   |  |
|  |                                                                   |  |
|  |  Example:                                                         |  |
|  |  "I'm working on a Python REST API using FastAPI.                 |  |
|  |   The project uses SQLAlchemy for database access and            |  |
|  |   follows the repository pattern. Authentication is via JWT."    |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                              |                                          |
|                              v                                          |
|  +-------------------------------------------------------------------+  |
|  |                         2. TASK                                   |  |
|  |                                                                   |  |
|  |  What you want the AI to do                                       |  |
|  |                                                                   |  |
|  |  Example:                                                         |  |
|  |  "Create a function that validates email addresses and            |  |
|  |   checks if the domain has valid MX records."                     |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                              |                                          |
|                              v                                          |
|  +-------------------------------------------------------------------+  |
|  |                      3. CONSTRAINTS                               |  |
|  |                                                                   |  |
|  |  Boundaries and preferences                                       |  |
|  |                                                                   |  |
|  |  Example:                                                         |  |
|  |  "Use only built-in modules (no external dependencies).           |  |
|  |   Handle None values gracefully. Return a tuple of                |  |
|  |   (is_valid: bool, error_message: str | None)."                   |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
+-------------------------------------------------------------------------+
```

### 1. Context

What the AI needs to know about your situation:
- What language/framework you're using
- What already exists
- Any constraints or requirements

### 2. Task

What you want the AI to do:
- Specific action (write, fix, refactor, explain)
- Clear scope (one function, one feature)

### 3. Constraints

Boundaries and preferences:
- Style requirements
- What NOT to do
- Performance considerations

### Example: Building a Prompt

**Poor prompt:**
> "Write a function to parse CSV"

**Good prompt:**
> "Write a Python function that parses a CSV file. The function should:
> - Take a file path as input
> - Return a list of dictionaries where keys are column headers
> - Handle missing values by setting them to None
> - Use the built-in csv module (no pandas)"

The difference: The good prompt provides context (Python, built-in csv), task (parse CSV), and constraints (no pandas, handle missing values).

---

## First Session Workflow

Before diving into code generation, understand the overall workflow for a successful AI coding session:

```
+-------------------------------------------------------------------------+
|                      FIRST SESSION WORKFLOW                             |
+-------------------------------------------------------------------------+
|                                                                         |
|  1. INSTALL & CONFIGURE                                                 |
|     +----------------------------------------------+                    |
|     |  claude login  /  Cursor setup               |                    |
|     |  Verify authentication works                 |                    |
|     +----------------------------------------------+                    |
|                           |                                             |
|                           v                                             |
|  2. NAVIGATE TO PROJECT                                                 |
|     +----------------------------------------------+                    |
|     |  cd /your/project                            |                    |
|     |  AI scans project structure                  |                    |
|     +----------------------------------------------+                    |
|                           |                                             |
|                           v                                             |
|  3. START SIMPLE                                                        |
|     +----------------------------------------------+                    |
|     |  Single function, clear prompt               |                    |
|     |  Use Context + Task + Constraints format     |                    |
|     +----------------------------------------------+                    |
|                           |                                             |
|                           v                                             |
|  4. REVIEW OUTPUT                                                       |
|     +----------------------------------------------+                    |
|     |  Read every line                             |                    |
|     |  Understand the approach                     |                    |
|     |  Check for red flags (see checklist below)   |                    |
|     +----------------------------------------------+                    |
|                           |                                             |
|                           v                                             |
|  5. TEST & VERIFY                                                       |
|     +----------------------------------------------+                    |
|     |  Run the code                                |                    |
|     |  Test edge cases                             |                    |
|     |  Verify behavior matches intent              |                    |
|     +----------------------------------------------+                    |
|                           |                                             |
|                           v                                             |
|  6. ITERATE                                                             |
|     +----------------------------------------------+                    |
|     |  Report issues to AI                         |                    |
|     |  Refine until satisfied                      |                    |
|     |  Only then: commit code                      |                    |
|     +----------------------------------------------+                    |
|                                                                         |
+-------------------------------------------------------------------------+
```

---

## Your First Code Generation

Let's generate code together. Follow along in your tool of choice.

### The Task

We'll create a function to validate email addresses.

### Step 1: Write the Prompt

In Claude Code or Cursor, enter:

```
Write a Python function called `is_valid_email` that:
- Takes a string as input
- Returns True if it's a valid email address, False otherwise
- Uses a regex pattern for validation
- Handles edge cases like empty strings and None values
- Include docstring with examples
```

### Step 2: Review the Generated Code

You might receive something like:

```python
import re

def is_valid_email(email: str) -> bool:
    """
    Validate if a string is a properly formatted email address.

    Args:
        email: The string to validate

    Returns:
        True if valid email format, False otherwise

    Examples:
        >>> is_valid_email("user@example.com")
        True
        >>> is_valid_email("invalid-email")
        False
        >>> is_valid_email("")
        False
        >>> is_valid_email(None)
        False
    """
    if email is None or not isinstance(email, str):
        return False

    if not email or email.isspace():
        return False

    # RFC 5322 simplified pattern
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

    return bool(re.match(pattern, email))
```

### Step 3: Understand Before Accepting

Before using this code, verify you understand it:

**Line by line:**
1. `import re` — Uses Python's regex module
2. Type hints indicate input (str) and output (bool)
3. Docstring documents behavior and gives examples
4. First check: handles None and non-string inputs
5. Second check: rejects empty strings and whitespace-only strings
6. Pattern: matches standard email format
7. `re.match` returns a match object or None; `bool()` converts to True/False

**Questions to ask yourself:**
- What emails would this accept? Reject?
- What's the regex pattern doing?
- Are there edge cases it misses?

### Step 4: Test the Code

Don't trust—verify:

```python
# Test cases
print(is_valid_email("user@example.com"))      # Expected: True
print(is_valid_email("user.name@domain.co.uk")) # Expected: True
print(is_valid_email("invalid"))               # Expected: False
print(is_valid_email(""))                      # Expected: False
print(is_valid_email(None))                    # Expected: False
print(is_valid_email("user@.com"))             # Expected: False
print(is_valid_email("@domain.com"))           # Expected: False
```

### Step 5: Iterate If Needed

If testing reveals issues, refine your prompt:

```
The email validation function incorrectly accepts "user@.com".
Please fix the regex to require at least one character before the dot in the domain.
```

---

## Multi-File Task Example

Real-world development involves multiple files working together. Here's an example of generating a small feature across model, service, and test files.

### The Request

```
Create a simple task management feature with:
1. A Task model with id, title, description, completed, and created_at fields
2. A TaskService with methods to create, list, and complete tasks
3. Unit tests for the TaskService

Use Python with dataclasses. Store tasks in memory (no database).
```

### AI-Generated Multi-File Response

**File 1: models/task.py**

```python
# models/task.py
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional
import uuid

@dataclass
class Task:
    """Represents a task in the task management system."""

    title: str
    description: str = ""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    completed: bool = False
    created_at: datetime = field(default_factory=datetime.now)

    def mark_completed(self) -> None:
        """Mark this task as completed."""
        self.completed = True

    def to_dict(self) -> dict:
        """Convert task to dictionary for serialization."""
        return {
            "id": self.id,
            "title": self.title,
            "description": self.description,
            "completed": self.completed,
            "created_at": self.created_at.isoformat(),
        }
```

**File 2: services/task_service.py**

```python
# services/task_service.py
from typing import List, Optional
from models.task import Task

class TaskService:
    """Service for managing tasks."""

    def __init__(self):
        self._tasks: dict[str, Task] = {}

    def create_task(self, title: str, description: str = "") -> Task:
        """
        Create a new task.

        Args:
            title: The task title (required)
            description: Optional task description

        Returns:
            The created Task object

        Raises:
            ValueError: If title is empty
        """
        if not title or not title.strip():
            raise ValueError("Task title cannot be empty")

        task = Task(title=title.strip(), description=description.strip())
        self._tasks[task.id] = task
        return task

    def list_tasks(self, include_completed: bool = True) -> List[Task]:
        """
        List all tasks.

        Args:
            include_completed: If False, only return incomplete tasks

        Returns:
            List of Task objects
        """
        tasks = list(self._tasks.values())
        if not include_completed:
            tasks = [t for t in tasks if not t.completed]
        return sorted(tasks, key=lambda t: t.created_at, reverse=True)

    def get_task(self, task_id: str) -> Optional[Task]:
        """Get a task by ID, or None if not found."""
        return self._tasks.get(task_id)

    def complete_task(self, task_id: str) -> Task:
        """
        Mark a task as completed.

        Args:
            task_id: The ID of the task to complete

        Returns:
            The updated Task object

        Raises:
            KeyError: If task not found
        """
        task = self._tasks.get(task_id)
        if task is None:
            raise KeyError(f"Task not found: {task_id}")
        task.mark_completed()
        return task
```

**File 3: tests/test_task_service.py**

```python
# tests/test_task_service.py
import pytest
from services.task_service import TaskService
from models.task import Task

class TestTaskService:
    """Tests for TaskService."""

    def setup_method(self):
        """Create a fresh TaskService for each test."""
        self.service = TaskService()

    def test_create_task_with_title_only(self):
        """Should create task with just a title."""
        task = self.service.create_task("Buy groceries")

        assert task.title == "Buy groceries"
        assert task.description == ""
        assert task.completed is False
        assert task.id is not None

    def test_create_task_with_description(self):
        """Should create task with title and description."""
        task = self.service.create_task(
            "Buy groceries",
            "Milk, eggs, bread"
        )

        assert task.title == "Buy groceries"
        assert task.description == "Milk, eggs, bread"

    def test_create_task_strips_whitespace(self):
        """Should strip whitespace from title and description."""
        task = self.service.create_task("  Buy groceries  ", "  items  ")

        assert task.title == "Buy groceries"
        assert task.description == "items"

    def test_create_task_empty_title_raises(self):
        """Should raise ValueError for empty title."""
        with pytest.raises(ValueError, match="cannot be empty"):
            self.service.create_task("")

        with pytest.raises(ValueError, match="cannot be empty"):
            self.service.create_task("   ")

    def test_list_tasks_returns_all(self):
        """Should return all tasks by default."""
        self.service.create_task("Task 1")
        self.service.create_task("Task 2")
        task3 = self.service.create_task("Task 3")
        self.service.complete_task(task3.id)

        tasks = self.service.list_tasks()

        assert len(tasks) == 3

    def test_list_tasks_exclude_completed(self):
        """Should filter out completed tasks when requested."""
        self.service.create_task("Task 1")
        task2 = self.service.create_task("Task 2")
        self.service.complete_task(task2.id)

        tasks = self.service.list_tasks(include_completed=False)

        assert len(tasks) == 1
        assert tasks[0].title == "Task 1"

    def test_complete_task_success(self):
        """Should mark task as completed."""
        task = self.service.create_task("Task 1")
        assert task.completed is False

        completed = self.service.complete_task(task.id)

        assert completed.completed is True
        assert completed.id == task.id

    def test_complete_task_not_found(self):
        """Should raise KeyError for unknown task ID."""
        with pytest.raises(KeyError, match="Task not found"):
            self.service.complete_task("nonexistent-id")

    def test_get_task_found(self):
        """Should return task by ID."""
        created = self.service.create_task("Task 1")

        found = self.service.get_task(created.id)

        assert found is not None
        assert found.id == created.id

    def test_get_task_not_found(self):
        """Should return None for unknown ID."""
        found = self.service.get_task("nonexistent-id")

        assert found is None
```

### Reviewing Multi-File Output

When you receive multi-file output, review each file systematically:

1. **Check the model:** Are the fields appropriate? Is the dataclass well-structured?
2. **Check the service:** Does it follow good patterns? Error handling present?
3. **Check the tests:** Do they cover success and failure cases? Edge cases?
4. **Check consistency:** Do files import each other correctly? Naming consistent?

---

## Debugging with AI Assistance

Beyond generating new code, AI assistants excel at debugging. Here's how to approach debugging effectively:

### The Debugging Workflow

1. **Reproduce the problem:** Ensure you can consistently trigger the bug
2. **Gather context:** Collect error messages, stack traces, and relevant code
3. **Present clearly to AI:** Structure your debugging request
4. **Verify the fix:** Test that the fix works without introducing new issues

### Effective Debugging Prompts

**Poor debugging prompt:**
> "It's not working, fix it"

**Better debugging prompt:**
> "This function throws an IndexError when the input list is empty:
>
> [paste the function]
>
> Error message:
> [paste the exact error]
>
> Please fix the function to handle empty lists gracefully by returning an empty result."

### Key Debugging Information to Include

| Information | Why It Matters |
|-------------|----------------|
| Exact error message | Tells AI precisely what went wrong |
| Stack trace | Shows where the error occurred |
| Input that caused the error | Helps reproduce and test the fix |
| Expected vs. actual behavior | Clarifies what "working" means |
| Code that failed | Context for understanding the bug |

### When AI Debugging Fails

If the AI's fix doesn't work:

1. **Report the failure:** Tell the AI what happened when you tried its fix
2. **Provide new information:** Include any new error messages or behaviors
3. **Ask for alternatives:** "That fix didn't work. What else could cause this?"
4. **Narrow the scope:** Ask for help with a smaller part of the problem

### Code Example: Debugging Workflow in Action

Here's a complete example of debugging with AI assistance.

**Starting Point: The Buggy Code**

```python
# shopping_cart.py - Has a bug!
from dataclasses import dataclass
from typing import List

@dataclass
class Item:
    name: str
    price: float
    quantity: int

class ShoppingCart:
    def __init__(self):
        self.items: List[Item] = []

    def add_item(self, name: str, price: float, quantity: int = 1):
        """Add an item to the cart."""
        for item in self.items:
            if item.name == name:
                item.quantity += quantity
                return
        self.items.append(Item(name, price, quantity))

    def calculate_total(self, discount_percent: float = 0) -> float:
        """Calculate the total with optional discount."""
        total = sum(item.price * item.quantity for item in self.items)
        discount = total * discount_percent / 100
        return total - discount

    def apply_bulk_discount(self) -> float:
        """Apply 10% discount if total > $100."""
        total = self.calculate_total()
        if total > 100:
            return self.calculate_total(discount_percent=10)
        return total


# Testing reveals a bug:
cart = ShoppingCart()
cart.add_item("Laptop", 999.99, 1)
cart.add_item("Mouse", 29.99, 2)

print(f"Subtotal: ${cart.calculate_total():.2f}")
print(f"With bulk discount: ${cart.apply_bulk_discount():.2f}")
print(f"Mouse only total: ${cart.calculate_total():.2f}")  # Bug shows here!

# Output shows something strange when we have the same item twice:
cart2 = ShoppingCart()
cart2.add_item("Book", 10, 1)
cart2.add_item("Book", 15, 1)  # Same name, different price!
print(f"Two books total: ${cart2.calculate_total():.2f}")
# Expected: $25.00 (10 + 15)
# Actual: $20.00 (10 * 2) - Bug! Second price was ignored
```

**Step 1: Present the Bug to AI**

A well-structured debugging prompt:

```
I have a bug in my ShoppingCart class. When adding the same item twice with
different prices, the second price is ignored.

Here's the buggy code:
[paste the ShoppingCart class above]

The bug:
- cart.add_item("Book", 10, 1)
- cart.add_item("Book", 15, 1)
- cart.calculate_total() returns $20.00
- Expected: $25.00 (should be 10 + 15)

The issue seems to be in add_item(), which matches only on name and ignores
the price when combining quantities. But I'm not sure what the right fix is.

Options I'm considering:
1. Always add as separate items
2. Match on name AND price
3. Reject items with same name but different prices

Which approach is best for a shopping cart, and can you implement it?
```

**Step 2: AI Provides Fix and Explanation**

```python
# Fixed shopping_cart.py
from dataclasses import dataclass
from typing import List

@dataclass
class Item:
    name: str
    price: float
    quantity: int

    @property
    def item_key(self) -> tuple:
        """Unique key for this item (name + price combination)."""
        return (self.name, self.price)

class ShoppingCart:
    def __init__(self):
        self.items: List[Item] = []

    def add_item(self, name: str, price: float, quantity: int = 1):
        """
        Add an item to the cart.

        If an item with the same name AND price exists, increment quantity.
        Otherwise, add as a new line item.

        This allows:
        - "Book" at $10 + "Book" at $10 = 1 item, quantity 2
        - "Book" at $10 + "Book" at $15 = 2 separate items
        """
        for item in self.items:
            if item.name == name and item.price == price:  # Match on BOTH
                item.quantity += quantity
                return
        self.items.append(Item(name, price, quantity))

    # Rest of the class unchanged...
```

**Step 3: Verify the Fix**

```python
# Test that the fix works
cart = ShoppingCart()
cart.add_item("Book", 10, 1)
cart.add_item("Book", 15, 1)
print(f"Two different books: ${cart.calculate_total():.2f}")
# Now outputs: $25.00

cart.add_item("Book", 10, 1)  # Same book as first
print(f"After adding another $10 book: ${cart.calculate_total():.2f}")
# Now outputs: $35.00 (two $10 books + one $15 book)
```

**Key Debugging Lessons:**

1. **Describe the bug specifically:** Input, expected output, actual output
2. **Show your analysis:** What you think might be wrong
3. **Propose options:** If you have ideas, share them for discussion
4. **Verify the fix:** Test with the original failing case AND new cases

---

## Test Generation with AI

AI assistants excel at generating test cases. Here's how to request comprehensive tests.

### The Test Generation Prompt

```
Write comprehensive pytest tests for this password validator function.
Cover: valid passwords, each validation rule failure, edge cases.

```python
def validate_password(password: str) -> tuple[bool, list[str]]:
    """
    Validate a password against security requirements.

    Returns:
        (is_valid, list_of_errors)
    """
    errors = []

    if len(password) < 8:
        errors.append("Password must be at least 8 characters")

    if len(password) > 128:
        errors.append("Password must not exceed 128 characters")

    if not any(c.isupper() for c in password):
        errors.append("Password must contain at least one uppercase letter")

    if not any(c.islower() for c in password):
        errors.append("Password must contain at least one lowercase letter")

    if not any(c.isdigit() for c in password):
        errors.append("Password must contain at least one number")

    if not any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?" for c in password):
        errors.append("Password must contain at least one special character")

    return (len(errors) == 0, errors)
```
```

### AI-Generated Tests

```python
# test_password_validator.py
import pytest
from password_validator import validate_password

class TestValidatePassword:
    """Comprehensive tests for password validation."""

    # ==================== VALID PASSWORDS ====================

    def test_valid_password_minimal(self):
        """Password meeting all requirements minimally."""
        is_valid, errors = validate_password("Abcdef1!")
        assert is_valid is True
        assert errors == []

    def test_valid_password_complex(self):
        """Complex password with multiple special chars."""
        is_valid, errors = validate_password("MyP@ssw0rd!Complex#2025")
        assert is_valid is True
        assert errors == []

    def test_valid_password_exactly_8_chars(self):
        """Password at minimum length boundary."""
        is_valid, errors = validate_password("Abc123!@")  # Exactly 8
        assert is_valid is True

    def test_valid_password_at_max_length(self):
        """Password at maximum length boundary."""
        password = "A" + "a" * 124 + "1!@"  # Exactly 128 chars
        is_valid, errors = validate_password(password)
        assert is_valid is True

    # ==================== LENGTH VALIDATION ====================

    def test_too_short_password(self):
        """Password under minimum length."""
        is_valid, errors = validate_password("Ab1!")
        assert is_valid is False
        assert "at least 8 characters" in errors[0]

    def test_too_long_password(self):
        """Password over maximum length."""
        password = "Aa1!" + "x" * 130  # Over 128 chars
        is_valid, errors = validate_password(password)
        assert is_valid is False
        assert "not exceed 128" in errors[0]

    def test_empty_password(self):
        """Empty string should fail multiple rules."""
        is_valid, errors = validate_password("")
        assert is_valid is False
        assert len(errors) >= 1  # Multiple errors expected

    # ==================== CHARACTER REQUIREMENTS ====================

    def test_missing_uppercase(self):
        """Password without uppercase letters."""
        is_valid, errors = validate_password("abcdef1!")
        assert is_valid is False
        assert any("uppercase" in e for e in errors)

    def test_missing_lowercase(self):
        """Password without lowercase letters."""
        is_valid, errors = validate_password("ABCDEF1!")
        assert is_valid is False
        assert any("lowercase" in e for e in errors)

    def test_missing_digit(self):
        """Password without numbers."""
        is_valid, errors = validate_password("Abcdefgh!")
        assert is_valid is False
        assert any("number" in e for e in errors)

    def test_missing_special_char(self):
        """Password without special characters."""
        is_valid, errors = validate_password("Abcdefg1")
        assert is_valid is False
        assert any("special character" in e for e in errors)

    # ==================== MULTIPLE FAILURES ====================

    def test_multiple_missing_requirements(self):
        """Password failing multiple rules returns all errors."""
        is_valid, errors = validate_password("abcd")  # Short, no upper, no digit, no special
        assert is_valid is False
        assert len(errors) >= 4

    # ==================== EDGE CASES ====================

    def test_password_with_spaces(self):
        """Spaces should be allowed in password."""
        is_valid, errors = validate_password("My Pass1!")
        assert is_valid is True

    def test_password_with_unicode(self):
        """Unicode characters in password."""
        is_valid, errors = validate_password("Paassword1!!")
        assert is_valid is True  # Should work, though unicode handling may vary

    def test_all_special_characters(self):
        """Test each special character is accepted."""
        special_chars = "!@#$%^&*()_+-=[]{}|;:,.<>?"
        for char in special_chars:
            password = f"Abcdefg1{char}"
            is_valid, errors = validate_password(password)
            assert is_valid is True, f"Failed for special char: {char}"
```

### Test Generation Best Practices

When asking AI to generate tests:

1. **Specify the testing framework:** pytest, unittest, jest, etc.
2. **Request specific categories:** Happy path, error cases, edge cases
3. **Mention boundaries:** Min/max values, empty inputs, null handling
4. **Ask for descriptive test names:** Makes failures easier to understand
5. **Review for coverage gaps:** AI might miss some edge cases

---

## The Feedback Loop

AI-assisted coding is iterative:

```
┌─────────────────────────────────────────┐
│                                         │
│   Prompt → Read → Understand → Refine   │
│      ↑                           │      │
│      └───────────────────────────┘      │
│                                         │
└─────────────────────────────────────────┘
```

### Prompt
Write a clear, specific request

### Read
Examine the generated code carefully

### Understand
Verify you know what it does

### Refine
Provide feedback to improve

This loop continues until you have code you understand and trust.

---

## Line-by-Line Review Habits

Develop the habit of reviewing AI code systematically:

### 1. Start at Imports

```python
import re        # ✓ Standard library - safe
import requests  # ⚠ External dependency - is this needed?
import os        # ⚠ System access - review usage carefully
```

### 2. Check Function Signatures

```python
def process_user_data(data: dict) -> list:
```
- Are types reasonable?
- Does the function name describe its purpose?
- Are parameters what you expected?

### 3. Trace Control Flow

Follow the logic:
- What happens with valid input?
- What happens with invalid input?
- Where could exceptions occur?

### 4. Look for Red Flags

| Red Flag | Why It Matters |
|----------|----------------|
| `eval()` or `exec()` | Code injection risk |
| SQL string concatenation | SQL injection risk |
| `pickle.loads()` | Deserialization attacks |
| Hardcoded credentials | Security exposure |
| `os.system()` | Command injection risk |
| No input validation | Various vulnerabilities |

### Red Flags Checklist Visual

```
+-------------------------------------------------------------------------+
|                    RED FLAGS IN AI-GENERATED CODE                       |
+-------------------------------------------------------------------------+
|                                                                         |
|  STOP AND VERIFY if you see any of these patterns:                      |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] eval() or exec()                                             |  |
|  |      Risk: Arbitrary code execution                               |  |
|  |      Example: eval(user_input)                                    |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] SQL string concatenation                                     |  |
|  |      Risk: SQL injection attacks                                  |  |
|  |      Example: f"SELECT * FROM users WHERE id = {user_id}"         |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] pickle.loads() on untrusted data                             |  |
|  |      Risk: Arbitrary code execution via deserialization           |  |
|  |      Example: pickle.loads(user_uploaded_file)                    |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] Hardcoded credentials                                        |  |
|  |      Risk: Credential exposure in source control                  |  |
|  |      Example: api_key = "sk-abc123..."                            |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] os.system() or subprocess with user input                    |  |
|  |      Risk: Command injection                                      |  |
|  |      Example: os.system(f"ls {user_path}")                        |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] innerHTML = userInput (JavaScript)                           |  |
|  |      Risk: Cross-site scripting (XSS)                             |  |
|  |      Example: element.innerHTML = userData.name                   |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [!] No input validation                                          |  |
|  |      Risk: Multiple attack vectors                                |  |
|  |      Example: Directly using user input without checks            |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +===================================================================+  |
|  ||                                                                 ||  |
|  ||   If you see ANY of these: STOP, understand the risk, and ask   ||  |
|  ||   the AI for a secure alternative before proceeding.            ||  |
|  ||                                                                 ||  |
|  +===================================================================+  |
|                                                                         |
+-------------------------------------------------------------------------+
```

### 5. Check Edge Cases

- Empty inputs
- None values
- Very large inputs
- Special characters
- Unicode

---

## Common First-Session Mistakes

### Mistake 1: Accepting Code Without Reading

**Problem:** Copying AI code directly into your project

**Solution:** Always read and understand before accepting

### Mistake 2: Vague Prompts

**Problem:** "Make a login system"

**Solution:** Be specific: "Create a function that validates a username and password against a SQLite database, using bcrypt for password hashing"

### Mistake 3: Not Testing

**Problem:** Assuming generated code works

**Solution:** Run the code with test cases before integrating

### Mistake 4: One-Shot Expectations

**Problem:** Expecting perfect code on the first try

**Solution:** Plan for iteration; use the feedback loop

### Mistake 5: Ignoring Error Messages

**Problem:** Giving up when code doesn't work

**Solution:** Paste error messages back to the AI for debugging help

---

## Key Takeaways

1. **Choose a tool** that fits your workflow (Claude Code for CLI, Cursor for IDE)
2. **Understand ACE vs. AEE:** Start with AI-assisted (ACE) mode before trying fully agentic (AEE) workflows
3. **Know your troubleshooting options:** Common issues have known solutions; consult the tables above
4. **Structure prompts** with context, task, and constraints
5. **Debug systematically:** Include error messages, code, and expected behavior when asking for help
6. **Review code line by line** before accepting
7. **Use the feedback loop**: prompt → read → understand → refine
8. **Test before trusting** — run code with various inputs
9. **Look for red flags** in security-sensitive code

---

## Practical Exercise: Build a CSV Parser

Put your new skills to practice:

### Task

Ask the AI to generate a CSV parser with these requirements:

1. Reads a CSV file from a given path
2. Returns a list of dictionaries
3. Handles quoted fields (fields containing commas)
4. Handles missing values
5. Includes error handling for file not found

### Process

1. **Write your prompt** (use the context/task/constraints structure)
2. **Review the generated code** line by line
3. **Identify anything you don't understand** and ask for explanation
4. **Create a test CSV file** and run the code
5. **Iterate** until it works correctly

### Reflection Questions

After completing the exercise:
- Could you explain this code to someone else?
- What edge cases might cause problems?
- What would you change if this were production code?

---

## Next Step

[Step 04: Effective Prompting for Code](04-effective-prompting-for-code.md) — Master the art of writing prompts that generate better code.

---

## Sources

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. https://arxiv.org/abs/2509.06216

- Anthropic: Claude Code Documentation. https://docs.anthropic.com/en/docs/claude-code
- Cursor: Official Documentation. https://docs.cursor.com/
- GitHub Copilot: Getting Started Guide. https://docs.github.com/en/copilot
- Anthropic: "Claude Code Best Practices." https://docs.anthropic.com/en/docs/claude-code/best-practices

---

## Further Reading

1. **Anthropic: Claude Code Best Practices** — Official guide to productive AI-assisted development. https://docs.anthropic.com/en/docs/claude-code/best-practices

2. **Cursor Documentation: Agent Mode** — Guide to using Cursor's autonomous coding features. https://docs.cursor.com/agent

3. **GitHub Copilot Documentation** — Comprehensive guide to using Copilot effectively. https://docs.github.com/en/copilot

4. **Windsurf (Codeium) Documentation** — Getting started with autonomous AI coding flows. https://docs.codeium.com/

5. **VS Code Extensions for AI Coding** — Overview of AI-assisted development tools for VS Code. https://code.visualstudio.com/docs/editor/artificial-intelligence
