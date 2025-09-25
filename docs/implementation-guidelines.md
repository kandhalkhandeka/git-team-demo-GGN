# Govern Chatmode User Guide

## Table of Contents
1. [How to Use](#how-to-use)
2. [Starting a Fresh Project](#starting-a-fresh-project)
3. [Extending an Existing Project](#extending-an-existing-project)
4. [Regenerating Components](#regenerating-components)
5. [Complete Project Rebuild](#complete-project-rebuild)
6. [Maintenance & Troubleshooting](#maintenance--troubleshooting)
7. [Common Development Patterns](#common-development-patterns)

---

## How to Use

1. **Place Generated Files:**
   - Copy the generated project files to your project's root directory
   - This includes the `metadata/`, `docs/`, `.github/`, and `.vscode/` folders

2. **Select Govern Chatmode:**
   - Open GitHub Copilot chat in VS Code
   - Select "Govern" chatmode from available modes

3. **Start Development:**
   - Use natural language to describe what you want to build
   - The agent will automatically detect metadata and execute tasks
   - A project tracker file will be created to monitor progress

### Adding Context to Your Requests
For better results, you can include specific files in your requests. Here are some examples:

**Include metadata files:**
```
"Generate React forms for scenario 930"
Add ./metadata/scenario_930.metadata.json for context.
```

**Include existing code:**
```
"Fix errors in my React components"
Add ./src/components/MyComponent.jsx for context.
```

**Include multiple files:**
```
"Integrate my frontend with backend API"
Add ./src/screens/IntakeForm.jsx and ./server/services/intake.py for context.
```

---

## Starting a Fresh Project

### When to Use This Approach
- No existing source code in your project
- Starting completely from scratch
- Want a clean, metadata-driven implementation

### Development Process

#### Step 1: Setup and Analysis
**Ask the Govern Agent:**
```
"Analyze my metadata and set up a new project with complete application structure"
```
You can add `./metadata/scenario_XXX.metadata.json` for context.
Include technology preferences, security requirements, and any workflow modifications from standard processes.

#### Step 2: Build Data Models
**Ask the Govern Agent:**
```
"Generate data models and validation schemas from my metadata"
```
You can add `./metadata/scenario_XXX.metadata.json` and `./metadata/pdm.json` for context.
Specify validation complexity, custom business rules, or integration requirements with existing systems.

#### Step 3: Create Frontend Components
**Ask the Govern Agent:**
```
"Create React forms and screens for my workflow with accessibility features"
```
You can add `./metadata/scenario_XXX.metadata.json` for context.
Include form behavior preferences, accessibility requirements, and styling preferences. Mention specific requirements for multi-step workflows and conditional fields.

#### Step 4: Build Backend Services
**Ask the Govern Agent:**
```
"Build secure API services with business rules and data validation"
```
You can add `./metadata/scenario_XXX.metadata.json` for context.
Include database preferences, authentication requirements, and security standards. Reference specific validation rules and workflow logic needed.

#### Step 5: Integration and Testing
**Ask the Govern Agent:**
```
"Connect frontend and backend components and validate the complete workflow"
```
Include error handling preferences and specific workflow testing scenarios.

---

## Extending an Existing Project

### When to Use This Approach
- You have working application but metadata has new requirements
- You want to add fields, validation, or workflow steps
- You need to preserve existing customizations and data

### Extension Process

#### Step 1: Analyze Current State
**Ask the Govern Agent:**
```
"Review my existing code and analyze what new requirements need to be added"
```
You can add `./metadata/scenario_XXX.metadata.json` and existing source files for context.
Mention specific areas of concern, performance issues, or new workflow requirements.

#### Step 2: Extend Frontend
**Ask the Govern Agent:**
```
"Add new fields and features to my existing forms while maintaining functionality"
```
You can add existing component files and updated metadata for context.
Reference specific components you're modifying and integration requirements.

#### Step 3: Enhance Backend
**Ask the Govern Agent:**
```
"Enhance my existing API with new endpoints and business rules"
```
You can add existing service files and metadata for context.
Specify new functionality and how it integrates with current backend architecture.

#### Step 4: Validate Changes
**Ask the Govern Agent:**
```
"Test my extended implementation for functionality and compliance"
```
Include specific compliance standards and integration testing scenarios.

---

## Regenerating Components

### When to Use This Approach
- Existing components don't match updated metadata structure
- You want to rebuild specific parts while keeping others
- Metadata changes are significant but you want to preserve customizations

### Regeneration Process

#### Step 1: Identify Changes
**Ask the Govern Agent:**
```
"Compare my current implementation with updated metadata and plan selective regeneration"
```
You can add current source files and updated `./metadata/scenario_XXX.metadata.json` for context.
Include which metadata sections changed and any custom code to preserve.

#### Step 2: Regenerate Components
**Ask the Govern Agent:**
```
"Regenerate my form components and data models based on updated workflow"
```
You can add `./metadata/scenario_XXX.metadata.json` and existing components for context.
Specify which workflow steps changed and customizations to preserve.

#### Step 3: Update Backend Logic
**Ask the Govern Agent:**
```
"Update my backend business rules to handle new requirements"
```
You can add `./metadata/scenario_XXX.metadata.json` and existing service files for context.
Reference specific validation logic or workflow steps that need updates.

#### Step 4: Test Regenerated System
**Ask the Govern Agent:**
```
"Test my regenerated components and ensure all functionality works correctly"
```
Include specific edge cases or integration scenarios critical for your deployment.

---

## Complete Project Rebuild

### When to Use This Approach
- Existing code is outdated or difficult to maintain
- Major metadata overhaul requires fresh start
- You want to take advantage of new framework features

### Rebuild Process

#### Step 1: Backup and Analysis
**Ask the Govern Agent:**
```
"Backup my current implementation and analyze metadata for a fresh rebuild"
```
You can add `./metadata/scenario_XXX.metadata.json` and existing project files for context.
Include lessons learned from previous implementation and new requirements.

#### Step 2: Setup New Foundation
**Ask the Govern Agent:**
```
"Set up a fresh project foundation with modern architecture and compliance features"
```
You can add `./metadata/scenario_XXX.metadata.json` for context.
Specify new technology preferences, architectural improvements, and performance requirements.

#### Step 3: Build Enhanced Data Layer
**Ask the Govern Agent:**
```
"Generate new data models and validation schemas with enhanced business rules"
```
You can add `./metadata/scenario_XXX.metadata.json` and `./metadata/pdm.json` for context.
Reference improvements needed from the previous data layer and enhanced validation requirements.

#### Step 4: Create Modern Frontend
**Ask the Govern Agent:**
```
"Build new React components with improved accessibility and modern UI patterns"
```
You can add `./metadata/scenario_XXX.metadata.json` for context.
Include modern UI/UX improvements and performance optimizations learned from previous implementation.

#### Step 5: Build Robust Backend
**Ask the Govern Agent:**
```
"Create new secure API services with enhanced business rules and improved logging"
```
You can add `./metadata/scenario_XXX.metadata.json` for context.
Specify improved database architecture and enhanced security patterns.

#### Step 6: Data Migration & Integration
**Ask the Govern Agent:**
```
"Implement data migration from the previous system and integrate new components"
```
You can add existing database schemas and data files for context.
Reference specific data structures and critical business data that must be migrated safely.

#### Step 7: Comprehensive Testing
**Ask the Govern Agent:**
```
"Perform comprehensive testing including data migration validation and performance testing"
```
Include validation of data migration accuracy and complete workflow testing.

---

## Maintenance & Troubleshooting

### When to Use This Approach
- Fixing bugs or performance issues
- Updating compliance requirements
- Regular maintenance and updates

### Maintenance Tasks

#### Fix Issues
**Ask the Govern Agent:**
```
"Help me debug and fix issues in my implementation"
```
You can add problematic source files and error logs for context.
Provide specific error messages, user feedback, or functionality problems you're experiencing.

#### Update Compliance
**Ask the Govern Agent:**
```
"Review and update my implementation for latest accessibility and security standards"
```
You can add current source files for context.
Specify new compliance requirements or security standards your organization has adopted.

#### Optimize Performance
**Ask the Govern Agent:**
```
"Analyze and improve the performance of my forms and backend processing"
```
You can add performance-critical source files for context.
Include specific performance issues or bottlenecks you've identified.

---

## Common Development Patterns

### Essential Context for All Requests
Always include relevant sections of your metadata files when making requests. The agent uses this to understand field types, validation rules, conditional logic, and workflow relationships.

### Technology Generated
The agent creates modern applications using:
- **Frontend:** React with form validation and multi-step workflows
- **Backend:** FastAPI with database integration and business rules
- **Database:** SQLite with proper data models
- **Architecture:** Production-ready, scalable application structure

### Including Multiple Files
For complex requests, include relevant portions of your existing code:
```
"Integrate my frontend forms with backend API"
You can add ./src/components/IntakeForm.jsx and ./server/services/intake.py for context.
```

### Advanced Field Types
When working with complex workflows, be aware of these field types in your metadata:
- **Textarea fields**: For narratives and comments
- **Radio button groups**: For yes/no selections
- **Auto-suggest fields**: For database lookups
- **Conditional visibility**: Fields that show/hide based on other selections
- **Repeatable sections**: Dynamic add/remove functionality

### Validation Patterns
Your system may include complex validation:
- **Cross-field validation**: Relationships between different fields
- **Conditional requirements**: Required fields based on other selections
- **Business rule enforcement**: Domain-specific validation logic
- **Format validation**: Phone numbers, email addresses, dates

### Project Tracker
The agent automatically creates a `project-tracker.json` file that:
- **Breaks down** your request into manageable tasks
- **Tracks progress** as each task completes
- **Resumes work** if interrupted - just ask the agent to continue
- **Shows status** of all development activities

You can check this file anytime to see what's been completed and what's next.

### Tips for Best Results
- **Be specific:** Mention scenario numbers when working with specific workflows
- **Use clear verbs:** "create", "build", "generate", "add", "fix"
- **Describe scope:** Specify if you want frontend, backend, or full application
- **Include context:** Add relevant metadata and source files to your requests
- **Let it work:** The agent handles all technical details automatically
- **Check progress:** Look at `project-tracker.json` to see completed tasks
- **Resume anytime:** If work stops, just ask "continue where you left off"

Simply describe what you want to build and include relevant files for context - the agent handles everything else!
