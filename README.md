# Govern Framework

A powerful metadata-driven application generator that transforms your workflow definitions into complete, production-ready web applications.

## What Is Govern?

Govern takes your business workflow metadata (JSON files) and automatically generates:
- **React frontend** with dynamic forms and multi-step workflows
- **FastAPI backend** with secure APIs and business rule engines
- **Database models** with validation and relationship management
- **Complete applications** ready for deployment

Simply describe your workflow in metadata files, then use natural language to generate your entire application.

## Quick Start

### 1. Setup Your Project
1. Copy all Govern framework files to your project's root directory
2. Ensure you have these folders in your project:
   ```
   your-project/
   ├── .github/
   ├── .vscode/
   ├── docs/
   ├── metadata/
   └── README.md (this file)
   ```

### 2. Add Your Workflow Metadata
Place your workflow definition files in the `metadata/` folder:
- `scenario_XXX.metadata.json` - Your workflow definitions
- `pdm.json` - Physical data model (optional)
- `scenario_map.json` - Multi-workflow orchestration (optional)

### 3. Start Building
1. Open your project in VS Code
2. Open GitHub Copilot chat
3. Select **"Govern"** from the chatmode dropdown
4. Start with a simple request:
   ```
   "Build a complete application from my metadata"
   ```

That's it! Govern will analyze your metadata and generate your entire application automatically.

> 📚 **For detailed usage patterns and examples, see [`docs/implementation-guidelines.md`](docs/implementation-guidelines.md)**

## What Gets Generated

### Frontend (React)
- **Dynamic forms** based on your field definitions
- **Multi-step workflows** using react-use-wizard
- **Validation logic** from your business rules
- **Responsive UI** with accessibility features
- **Navigation system** connecting all screens

### Backend (FastAPI)
- **REST API endpoints** for all your data operations
- **Business rule engines** implementing your validation logic
- **Database integration** with automatic model creation
- **Security features** including authentication and authorization
- **Background job processing** for complex workflows

### Database
- **SQLAlchemy models** matching your metadata structure
- **Automatic table creation** with proper relationships
- **Data validation** enforcing your business rules
- **Migration support** for schema updates

## Example Usage

### Basic Application Generation
```
"Create a complete application for scenario 930"
```
Add your `metadata/scenario_930.metadata.json` file for context.

### Adding New Features
```
"Add new fields to my intake form and update the backend"
```
Include your existing component files for context.

### Multi-Step Workflows
```
"Create a multi-step workflow with validation between steps"
```
Govern automatically detects multi-step scenarios and uses react-use-wizard.

### Business Rules
```
"Implement complex validation rules and calculations"
```
Converts your metadata validation arrays into working business logic.

### Background Processing
```
"Add background job processing for data analysis"
```
Creates Prefect workflows for heavy computation tasks.

## Technology Stack

Govern generates modern, production-ready applications using:

- **Frontend**: React with functional components and hooks
- **State Management**: react-use-wizard for multi-step flows
- **Backend**: FastAPI with automatic API documentation
- **Database**: SQLite (development) / MariaDB (production)
- **ORM**: SQLAlchemy with automatic model generation
- **Validation**: Pydantic schemas with business rule enforcement
- **Background Jobs**: Prefect for workflow automation
- **Authentication**: JWT-based security system

## Metadata Structure

Your workflow metadata should define:

### Fields
```json
{
  "fields": [
    {
      "name": "applicant_name",
      "type": "text",
      "label": "Applicant Name",
      "required": true,
      "validation": ["not_empty"]
    }
  ]
}
```

### Steps
```json
{
  "steps": [
    {
      "type": "Screen",
      "name": "intake_screen",
      "title": "Application Intake"
    }
  ]
}
```

### Business Rules
```json
{
  "data_validation": [
    {
      "field": "income",
      "rule": "greater_than",
      "value": 0
    }
  ]
}
```

## Advanced Features

### Context-Aware Generation
Include specific files in your requests for better results:
```
"Update my React components with new validation"
Add ./src/components/IntakeForm.jsx for context.
```

### Project Modes
- **Fresh Project**: Start from scratch with clean implementation
- **Enhancement**: Add features to existing applications  
- **Regeneration**: Rebuild components with updated metadata
- **Maintenance**: Fix issues and optimize performance

### Intelligent Detection
- Automatically detects your metadata structure and workflow type
- Selects appropriate generation patterns based on your requirements
- Handles complex relationships between multiple workflow scenarios
- Manages dependencies between frontend and backend components

## Project Organization

Generated applications follow this structure:
```
your-application/
├── src/                 # React frontend
│   ├── components/      # Reusable UI components
│   ├── screens/         # Workflow screens
│   └── services/        # API integration
├── server/              # FastAPI backend
│   ├── models/          # Database models
│   ├── services/        # Business logic
│   └── main.py          # Application entry point
├── metadata/            # Your workflow definitions
└── project-tracker.json # Progress tracking (auto-generated)
```

## Getting Help

### Implementation Guide
📖 **See [`docs/implementation-guidelines.md`](docs/implementation-guidelines.md)** for complete usage instructions including:
- Step-by-step development processes
- Context file inclusion examples  
- Different project implementation approaches
- Common request patterns and examples

### Built-in Guidance
- Check `docs/implementation-guidelines.md` for detailed usage patterns
- Review `project-tracker.json` to see progress and next steps
- Use natural language to ask for help: "How do I add validation to my forms?"

### Common Requests
- **"Analyze my metadata and show me what can be built"**
- **"Create React forms with accessibility features"**
- **"Build secure API endpoints with business rules"**
- **"Fix errors in my generated components"**
- **"Optimize performance of my application"**

### Best Practices
- Be specific about what you want to build
- Include relevant metadata and source files for context
- Use clear action words: "create", "build", "add", "fix", "optimize"
- Let Govern handle technical details automatically
- Check the project tracker to see what's been completed

## Workflow Types Supported

Govern works with any workflow domain:
- **Child Welfare**: Case management, assessments, reporting
- **Benefits Administration**: SNAP, TANF, Medicaid applications
- **Healthcare**: Patient intake, treatment planning, compliance
- **Government Services**: Permit applications, license renewals
- **Business Processes**: Any structured workflow with forms and rules

## Quality & Standards

All generated code includes:
- **Comprehensive error handling** with user-friendly messages
- **Accessibility compliance** following WCAG guidelines
- **Security best practices** including input validation and authentication
- **Performance optimization** with efficient database queries
- **Testing capabilities** with unit and integration test patterns
- **Documentation** for all components and APIs

## Deployment Ready

Generated applications are production-ready with:
- **Docker configuration** for containerized deployment
- **Environment configuration** for different deployment stages
- **Database migration** support for schema updates
- **Monitoring integration** for performance tracking
- **Logging system** for debugging and audit trails

## Support

Govern provides continuous assistance:
- **Automatic progress tracking** shows what's completed and what's next
- **Error recovery** attempts to fix issues automatically
- **Resume capability** continues work if interrupted
- **Validation checks** ensure generated code works correctly
- **Integration testing** verifies all components work together

---

**Ready to transform your workflow metadata into a complete application?**

Just open VS Code, select the Govern chatmode, and describe what you want to build!