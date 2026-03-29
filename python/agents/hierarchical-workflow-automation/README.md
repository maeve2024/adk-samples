# Cookie Delivery Agent System

A multi-agent system built with Google ADK that automates cookie delivery order processing, scheduling, and customer communication. The system integrates with BigQuery for order management, Google Calendar for delivery scheduling, and Gmail for customer notifications.

## Architecture Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Root Agent    │───►│ Sequential Agent │───►│  Sub-Agents     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                                         │
                                                         ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ Database Agent  │    │  Calendar Agent  │    │   Email Agent   │
│ (BigQuery ADK)  │    │    MCP Server    │    │ (LangChain API) │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│    BigQuery     │    │ Google Calendar  │    │     Gmail       │
│   (GCP Acct)    │    │  (Business Acct) │    │ (Business Acct) │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

### Agent Workflow

1. **Database Agent**: Fetches new orders from BigQuery using Google's first-party ADK toolset with status "order_placed"
2. **Calendar Agent**: Checks availability and schedules delivery appointments via MCP server
3. **Email Agent**: Generates personalized confirmation emails using LangChain Community Gmail toolkit and updates order status in BigQuery

## Setup Instructions

> ** Important for Argolis Users**: If you're using an Argolis corporate account, Gmail and Calendar access are restricted due to Google security policies. You'll need to create or use a **free secondary Gmail account** for Calendar and Gmail integration. While you can use Outlook or other services, these instructions are designed around Google Workspace/Gmail.

## Setup and Installation

### Prerequisites

- Python 3.10+
- uv for dependency management and packaging
  - See the official [uv website](https://docs.astral.sh/uv/) for installation.

  ```bash
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```

## Agent Starter Pack (recommended)

Use the [Agent Starter Pack](https://goo.gle/agent-starter-pack) to scaffold a production-ready project and choose your deployment target ([Vertex AI Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview) or [Cloud Run](https://cloud.google.com/run)), with CI/CD and other production features. The easiest way is with [uv](https://docs.astral.sh/uv/) (one command, no venv or pip install needed):

```bash
uvx agent-starter-pack create my-cookie-delivery -a adk@hierarchical-workflow-automation
```

If you don't have uv yet: `curl -LsSf https://astral.sh/uv/install.sh | sh`

The starter pack will prompt you to select deployment options and set up your Google Cloud project.

<details>
<summary>Alternative: Using pip and a virtual environment</summary>

```bash
# Create and activate a virtual environment
python -m venv .venv && source .venv/bin/activate # On Windows: .venv\Scripts\activate

# Install the starter pack and create your project
pip install --upgrade agent-starter-pack
agent-starter-pack create my-cookie-delivery -a adk@hierarchical-workflow-automation
```

</details>

From your newly created project directory (e.g. `my-cookie-delivery`), run:

```bash
cd my-cookie-delivery
uv sync --dev
uv run adk run cookie_scheduler_agent
```

For the web UI:

```bash
uv run adk web
```

Then select `cookie_scheduler_agent` from the dropdown menu.

---

<details>
<summary>Alternative: Local development (run from this sample repo)</summary>

### Agent Setup

1. Clone the repository:

   ```bash
   git clone https://github.com/google/adk-samples.git
   cd adk-samples/python/agents/hierarchical-workflow-automation
   ```

   For the rest of this tutorial **ensure you remain in the `python/agents/hierarchical-workflow-automation` directory**.

2. Install the dependencies:

   ```bash
   uv sync --dev
   ```

3. Configure settings:

   Copy the environment template and fill in your values:

   ```bash
   cp .env-template .env
   # Edit .env with your configuration
   ```

   Authenticate your GCloud account:

   ```bash
   gcloud auth application-default login
   gcloud auth application-default set-quota-project $GOOGLE_CLOUD_PROJECT
   ```

### Running the Agent Locally

You can run the agent locally using the `adk` command in your terminal.

1. To run the agent from the CLI:

   ```bash
   uv run adk run cookie_scheduler_agent
   ```

2. To run the agent from the ADK web UI:

   ```bash
   uv run adk web
   ```

   Then select `cookie_scheduler_agent` from the dropdown menu.

### Development

```bash
uv sync --dev
uv run pytest tests
```

</details>

## Environment Setup

> **Important for Argolis Users**: If you're using an Argolis corporate account, Gmail and Calendar access are restricted due to Google security policies. You'll need to create or use a **free secondary Gmail account** for Calendar and Gmail integration. While you can use Outlook or other services, these instructions are designed around Google Workspace/Gmail.

Copy the included template and fill in your values:

```bash
cp .env-template .env
# Edit .env with your project details
```

### Global Authentication

```bash
gcloud auth application-default login
gcloud config set project YOUR_PROJECT_ID
```

### Enable Required APIs

1. **BigQuery API**: Navigate to Google Cloud Console → "APIs & Services" → "Library" → Search "BigQuery API" → Enable
2. **Calendar API**: Same steps for "Google Calendar API"
3. **Gmail API**: Same steps for "Gmail API"

### Calendar & Gmail Credentials

> **For Argolis Users**: Use your **secondary Gmail account** for credential setup, not your corporate account.

1. Go to Google Cloud Console → "APIs & Services" → "Credentials"
2. Click "Create Credentials" → "OAuth 2.0 Client ID" → "Desktop Application"
3. Download the JSON file
4. Save as `cookie_scheduler_agent/mcp_servers/calendar/calendar_credentials.json` for Calendar
5. Save as `cookie_scheduler_agent/gmail_langchain/gmail_credentials.json` for Gmail

### BigQuery Setup

```bash
# Enable BigQuery Integration in .env file:
USE_BIGQUERY=true

# Optional: Create sample data for testing
uv run python cookie_scheduler_agent/bigquery_utils/create_bigquery_environment.py
```

## BigQuery Schema

The system creates the following BigQuery structure:

### Dataset: `cookie_delivery`
### Table: `orders`

```sql
CREATE TABLE `{PROJECT_ID}.cookie_delivery.orders` (
  order_id STRING NOT NULL,
  order_number STRING NOT NULL,
  customer_email STRING NOT NULL,
  customer_name STRING NOT NULL,
  customer_phone STRING,
  order_items ARRAY<STRUCT<
    item_name STRING,
    quantity INT64,
    unit_price FLOAT64
  >>,
  delivery_address STRUCT<
    street STRING,
    city STRING,
    state STRING,
    zip_code STRING,
    country STRING
  >,
  delivery_location STRING,
  delivery_request_date DATE,
  delivery_time_preference STRING,
  order_status STRING NOT NULL,
  total_amount FLOAT64,
  order_date TIMESTAMP,
  special_instructions STRING,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

## Current Implementation

### BigQuery ADK Integration
- **Google's First-Party ADK Toolset**: Uses official BigQuery ADK integration
- **Application Default Credentials**: Secure authentication via ADC
- **Available Tools**: list_dataset_ids, get_dataset_info, list_table_ids, get_table_info, execute_sql, ask_data_insights

### Gmail LangChain Integration
- **LangChain Community Gmail Toolkit**: Uses official LangChain integration for Gmail API
- **OAuth2 Authentication**: Secure authentication with automatic token refresh
- **HTML Email Support**: Rich formatting for professional customer communications

### Calendar Agent with Real Google Calendar
- **Real Google Calendar Integration**: Creates actual calendar events via MCP server
- **Smart Fallback**: Uses dummy data when Calendar MCP unavailable
- **RFC3339 Datetime**: Proper timezone handling for Google Calendar API

### Agent Workflow (Sequential Processing)
1. **Database Agent**: Fetches orders using BigQuery ADK toolset
2. **Calendar Agent**: Real Google Calendar scheduling via MCP server
3. **Email Agent**: Professional email communications via LangChain Gmail toolkit
4. **Haiku Writer Sub-Agent**: Generates creative seasonal content

### Error Handling & Resilience
- **Graceful Degradation**: Falls back to dummy data when services unavailable
- **Comprehensive Logging**: Detailed operation tracking and error reporting
- **Authentication Recovery**: Handles OAuth2 token refresh automatically

## Testing & Validation

### BigQuery ADK Testing

```bash
uv run python cookie_scheduler_agent/bigquery_utils/test_adk_bigquery_unit.py
uv run python cookie_scheduler_agent/bigquery_utils/test_adk_integration.py
```

### Calendar MCP Testing

```bash
uv run python cookie_scheduler_agent/mcp_servers/calendar/test_calendar_functions.py
```

## File Structure

```
hierarchical-workflow-automation/
├── .env.example              # Example environment configuration
├── requirements.txt          # Python dependencies
├── setup.sh                  # enables GCP APIs and permissions
├── deploy_agent.py           # Agent Engine deployment script
└── cookie_scheduler_agent/
    ├── agent.py                    # main adk agent orchestration
    ├── dummy_data.py              # Fallback data for testing
    │
    ├── gmail_langchain/
    │   ├── gmail_manager.py             # Main LangChain Gmail manager class
    │   ├── email_utils.py               # Utility functions
    │   ├── test_gmail_integration.py    # Test script
    │   └── README.md                    # Setup documentation
    │
    ├── bigquery_utils/           # BigQuery ADK toolset integration
    │   ├── bigquery_tools.py     # ADK BigQuery toolset implementation
    │   ├── create_bigquery_environment.py # BigQuery setup script
    │   ├── test_adk_bigquery_unit.py      # Unit tests
    │   ├── test_adk_integration.py        # Integration tests
    │   └── README.md                      # Directory documentation
    │
    └── mcp_servers/              # MCP Server implementations
        ├── calendar/             # Calendar MCP
        │   ├── calendar_mcp_server.py      # CalendarManager class
        │   └── test_calendar_functions.py  # Test script
        ├── start_calendar_mcp.py           # MCP server startup script
        └── setup_calendar_credentials.md   # Setup instructions
```

## Security Notes

- Never commit `.env`, `*_credentials.json`, or `*_token.json` files to version control
- Use Google Secret Manager for production deployments
- Implement credential rotation policies
- Use minimal required scopes for OAuth2

## Troubleshooting

### Calendar MCP Issues

1. **OAuth2 Authentication Failed**: Ensure Calendar API is enabled, create OAuth 2.0 Client ID (Desktop Application), delete `calendar_token.json` to force re-authentication
2. **Calendar Events Not Appearing**: Check `BUSINESS_CALENDAR_ID` in `.env`
3. **Permissions Error**: Ensure "External" user type in OAuth consent screen, add secondary Gmail as test user

### BigQuery ADK Issues

1. **Import Error**: Ensure `google-adk` package is installed
2. **Permission Denied**: Ensure BigQuery Data Editor and Job User roles
3. **403 Errors**: Verify correct project with `gcloud config set project <PROJECT_ID>`

### Debug Mode

Enable detailed logging:

```bash
# In .env file
LOG_LEVEL=DEBUG
```

## Support

For issues and questions:
1. Check the troubleshooting section above
2. Review the detailed setup docs in [bigquery_utils/README.md](./cookie_scheduler_agent/bigquery_utils/README.md), [gmail_langchain/README.md](./cookie_scheduler_agent/gmail_langchain/README.md), [mcp_servers/README.md](./cookie_scheduler_agent/mcp_servers/README.md)
3. Create an issue in the repository
