# DOMAIN MODEL GENERATION - SQLAlchemy Schema Development

## Command Patterns
When user inputs any of:
- `create models from metadata` 
- `generate domain models`
- `create SQLAlchemy models`
- `build data models`
- `extend existing models`
- `regenerate models`

## Purpose
Generate comprehensive SQLAlchemy models with Pydantic schemas based on metadata field specifications, following established project patterns with complete CRUD operations and TableManager integration.

## Prerequisites
- Metadata files analyzed and requirements extracted
- PDM (Physical Data Model) located and analyzed from `./metadata/pdm.json`
- Project technology stack configured (React + FastAPI + SQLAlchemy + Pydantic)
- Existing project patterns documented
- Database integration requirements understood

## Mode Detection
Automatically detect execution mode based on context:
- **create-models**: Fresh model generation from metadata and PDM requirements
- **extend-models**: Add new fields and functionality to existing models
- **regenerate-models**: Rebuild models while preserving customizations

---

## DISCOVERY AND ANALYSIS - Model Architecture Analysis

### Architecture Pattern Discovery - Schema Pattern Recognition
Analyze existing codebase to understand established patterns:

1. **Examine Documentation**
   - Check `README.md` and `docs/README.md` (if available) for database schema decisions and model conventions
   - Analyze PDM Requirements: Load and parse `./metadata/pdm.json` for complete entity definitions, field specifications, data types, and relationship mappings
   - Search workspace for architecture documentation files
   - Extract model creation patterns and naming conventions
   - Identify integration requirements and established guidelines
   - Document SQLAlchemy model development standards

2. **Analyze Existing Models**
   - Review all files in backend/models/ directory
   - PDM Compliance Check: Compare existing models against PDM entity definitions
   - Gap Analysis: Identify missing entities, fields, or relationships from PDM
   - Schema Validation: Flag discrepancies between current implementation and PDM requirements
   - Extract model structure patterns and field type conventions
   - Identify Pydantic schema patterns and naming standards
   - Document relationship and foreign key patterns
   - Collect best practices from existing implementations

3. **Study TableManager Integration**
   - Locate and analyze TableManager implementation
   - Understand model registration in _import_all_models() method
   - Document database integration patterns and table creation system
   - Extract import patterns and registration requirements

4. **Process Metadata Requirements**
   - Parse metadata field specifications and data types
   - Map PDM Entities: Cross-reference PDM entities with metadata workflow requirements
   - Data Type Alignment: Ensure metadata field types align with PDM field specifications
   - Map metadata field types to SQLAlchemy column types using PDM data type definitions
   - Extract validation rules and constraints from data_validation arrays
   - Identify relationships and database design requirements from PDM relationship mappings
   - Document business rules and calculation logic

### Consolidation and Planning
Create comprehensive implementation approach:
- PDM Integration Strategy: Plan implementation that follows PDM entity specifications exactly
- Migration Requirements: Identify database changes needed to align with PDM schema
- Identify conflicts between existing patterns and new requirements
- Plan integration strategy that maintains consistency
- Design backward-compatible extensions for existing models
- Establish testing and validation approach
---

## PDM (PHYSICAL DATA MODEL) VALIDATION

### PDM Analysis and Integration
Before generating any models, perform comprehensive PDM validation:

#### 1. **PDM Schema Loading**
Load and parse the Physical Data Model from `./metadata/pdm.json`:

```python
# Load PDM for comprehensive entity analysis
import json

with open('./metadata/pdm.json', 'r') as pdm_file:
    pdm_schema = json.load(pdm_file)

# Extract entity definitions
entities = pdm_schema.get('entities', [])
relationships = pdm_schema.get('relationships', [])
```

#### 2. **Entity Mapping Validation**
For each PDM entity, validate against existing models:

**Entity Structure Analysis:**
- **Name**: Extract entity name and verify naming conventions
- **Description**: Use for model documentation and comments
- **Fields**: Complete field mapping with data types, constraints, keys
- **Relationships**: Foreign key relationships and cardinality

**Field Specification Mapping:**
- **data_type**: Map to SQLAlchemy column types (VARCHAR → String, INTEGER → Integer)
- **required**: Map to nullable parameter (required=true → nullable=False)
- **unique**: Map to unique constraint
- **primary_key**: Identify primary key fields
- **foreign_key**: Extract foreign key relationships

#### 3. **Gap Analysis Report**
Generate comprehensive comparison between current implementation and PDM requirements:

**Missing Entities Report:**
```
Entities in PDM but missing from current implementation:
- Entity: SupervisorAvailability
  - Purpose: Tracks supervisor availability and capacity
  - Required Fields: availability_id, supervisor_id, availability_date, is_available
  - Dependencies: Requires Person entity relationship
```

**Field Discrepancies Report:**
```
Entity: Case
- Missing Fields: 
  - tags (JSON type) - for case categorization
  - referral_source (VARCHAR 255) - source of case referral
- Type Mismatches:
  - priority_level: Current=String(20), PDM=String(50)
  - risk_level: Current=String(20), PDM=String(50)
```

**Relationship Gaps Report:**
```
Missing Relationships:
- Person → SupervisorAvailability (one-to-many)
- Case → Document (one-to-many) 
- Person → Configuration (one-to-many for modified_by_id)
```

#### 4. **Migration Strategy Planning**
Based on gap analysis, create migration approach:

**Database Migration Requirements:**
- New table creation scripts for missing entities
- Column addition for missing fields
- Index creation for new unique/foreign key constraints
- Data migration scripts for existing data preservation

**Backward Compatibility Assessment:**
- Identify breaking changes that affect existing APIs
- Plan graceful migration strategies for production systems
- Document rollback procedures for failed migrations

#### 5. **PDM-Compliant Model Generation**
Generate models that exactly match PDM specifications:

```python
# Example: Generate model based on PDM entity definition
def generate_model_from_pdm_entity(entity_def):
    class_name = entity_def['name']
    table_name = entity_def.get('table_name', class_name.lower() + 's')
    
    # Generate SQLAlchemy columns based on PDM field definitions
    columns = []
    for field in entity_def['fields']:
        column_type = map_pdm_type_to_sqlalchemy(field['data_type'])
        column_def = f"{field['name']} = Column({column_type}"
        
        if field.get('primary_key'):
            column_def += ", primary_key=True"
        if not field.get('required', True):
            column_def += ", nullable=True"
        if field.get('unique'):
            column_def += ", unique=True"
        if field.get('foreign_key'):
            fk_table = field['foreign_key']
            column_def += f", ForeignKey('{fk_table.lower()}.id')"
            
        columns.append(column_def + ")")
    
    return generate_model_class(class_name, table_name, columns)
```

#### 6. **Existing Codebase Validation Workflow**
When an existing codebase is detected, perform comprehensive PDM validation:

**Step 1: Model Discovery and Analysis**
```python
# Discover existing models
existing_models = file_search("backend/models/*.py")
existing_entities = extract_model_definitions(existing_models)

# Load PDM requirements
pdm_entities = load_pdm_entities("./metadata/pdm.json")

# Compare and analyze
comparison_report = compare_existing_vs_pdm(existing_entities, pdm_entities)
```

**Step 2: Gap Analysis Report Generation**
```markdown
## PDM Compliance Report

### Missing Entities (require creation):
- **SupervisorAvailability**: Tracks supervisor capacity and availability
- **Configuration**: System settings and parameters
- **Activity**: Audit trail for system changes

### Missing Fields (require addition):
**Case Entity:**
- tags (JSON): Missing categorization field
- referral_source (VARCHAR 255): Missing source tracking
- priority_level size: Current=20, Required=50

**Person Entity:**
- availability_status (VARCHAR 50): Missing availability tracking
- max_caseload_capacity (INTEGER): Missing capacity management

### Type Mismatches (require correction):
- Case.screening_comments: Current=VARCHAR(500), PDM=TEXT
- Person.department: Current=VARCHAR(50), PDM=VARCHAR(100)

### Missing Relationships (require addition):
- Person → SupervisorAvailability (one-to-many)
- Case → Document (one-to-many)
- Person → Configuration (one-to-many)
```

**Step 3: Migration Strategy Planning**
```python
# Generate migration plan
migration_plan = {
    "new_entities": ["SupervisorAvailability", "Configuration", "Activity"],
    "field_additions": {
        "Case": ["tags", "referral_source"],
        "Person": ["availability_status", "max_caseload_capacity"]
    },
    "type_changes": {
        "Case.screening_comments": "VARCHAR(500) → TEXT",
        "Person.department": "VARCHAR(50) → VARCHAR(100)"
    },
    "new_relationships": [
        "Person → SupervisorAvailability",
        "Case → Document", 
        "Person → Configuration"
    ]
}
```

**Step 4: Backward-Compatible Implementation**
Generate implementation that preserves existing functionality:
- Add new entities with proper relationships
- Extend existing models with nullable fields (safe defaults)
- Create migration scripts for type changes
- Maintain API compatibility for existing endpoints

---

## MODEL GENERATION EXECUTION - SQLAlchemy Model Development

### SQLAlchemy Model Creation
Generate complete domain models following discovered patterns:

#### Core Model Structure
Create models with established inheritance patterns:

```python
from sqlalchemy import Column, Integer, String, DateTime, Boolean, Text, ForeignKey, Numeric
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime
from models.base import Base

class {ModelName}(Base):
    __tablename__ = "{table_name}"
    __table_args__ = {'extend_existing': True}
    
    # Standard primary key
    id = Column(Integer, primary_key=True, index=True)
    
    # Generated fields from metadata specifications
    {generated_fields_based_on_metadata}
    
    # Standard timestamp fields
    created_at = Column(DateTime(timezone=True), server_default=func.now(), nullable=False)
    updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False)
    
    # Relationships based on metadata requirements
    {generated_relationships}
```

#### Pydantic Schema Generation
Create complete CRUD schema set following project patterns:

```python
# Base schema with shared fields
class {ModelName}Base(BaseModel):
    {generated_base_fields_with_validation}

# Creation schema
class {ModelName}Create({ModelName}Base):
    pass

# Update schema with optional fields
class {ModelName}Update(BaseModel):
    {generated_optional_fields}

# Response schema with all fields
class {ModelName}Response({ModelName}Base):
    id: int
    created_at: datetime
    updated_at: datetime
    {additional_response_fields}
    
    class Config:
        from_attributes = True
```

### Dynamic Field Generation
Map metadata field types to SQLAlchemy columns:

- **text/textarea** → `Column(String(length), nullable=constraint)`
- **dropdown** → `Column(String(50), nullable=False)` with validation
- **number** → `Column(Integer, nullable=constraint)` with validation
- **currency** → `Column(Numeric(precision=10, scale=2), nullable=constraint)`
- **boolean** → `Column(Boolean, default=value, nullable=False)`
- **date** → `Column(DateTime(timezone=True), nullable=constraint)`
- **readOnly: true** → Calculated field with proper typing

### Business Rules Implementation
Transform metadata validation rules into Pydantic validation:

1. **Extract validation constraints** from data_validation arrays
2. **Generate Field() validators** with proper constraints and descriptions
3. **Implement cross-field validation** for complex business rules
4. **Create calculation logic** for readonly/computed fields
5. **Generate user-friendly error messages** from metadata specifications

### TableManager Integration
Ensure proper model registration:

```python
# Add to TableManager._import_all_models() method
from models.{ModelName} import {ModelName}
```

### Relationship Handling
Generate relationships based on metadata specifications:
- **Foreign Keys**: `Column(Integer, ForeignKey('table.id'))`
- **One-to-Many**: `relationship("RelatedModel", back_populates="parent")`
- **Many-to-One**: `relationship("ParentModel", back_populates="children")`
- **Many-to-Many**: Through association tables when specified

---

## EXTEND-MODELS MODE - Model Enhancement and Extension

### Existing Model Analysis
When extending existing models:

1. **Analyze Current Model Structure**
   - Review existing fields and their types
   - PDM Compliance Audit: Compare current models against PDM entity requirements
   - Missing Field Detection: Identify fields from PDM that are missing in current implementation
   - Type Validation: Verify current field types match PDM specifications
   - Document current Pydantic schemas
   - Identify existing relationships and constraints
   - Check current TableManager registration

2. **Plan Backward-Compatible Extensions**
   - PDM Alignment: Plan additions to bring models into PDM compliance
   - Add new fields with appropriate defaults based on PDM specifications
   - Extend Pydantic schemas without breaking existing APIs
   - Plan database migration strategy for new columns
   - Ensure existing functionality remains intact

### Extension Implementation
Safely extend existing models:

```python
# Add new fields to existing model
class ExistingModel(Base):
    # ... existing fields remain unchanged ...
    
    # New fields with safe defaults
    new_field = Column(String(100), nullable=True, default=None)
    new_calculated_field = Column(Numeric(10,2), nullable=True)
    
    # Extended relationships if needed
    new_relationship = relationship("NewModel", back_populates="parent")
```

Update Pydantic schemas:
```python
# Extend base schema with new fields
class ExistingModelBase(BaseModel):
    # ... existing fields ...
    new_field: Optional[str] = Field(None, max_length=100, description="New field description")

# Update other schemas accordingly
class ExistingModelUpdate(BaseModel):
    # ... existing optional fields ...
    new_field: Optional[str] = Field(None, max_length=100, description="New field description")
```

---

## REGENERATE-MODELS MODE - Schema Rebuilding with Preservation

### Preservation Strategy
**Purpose**: Rebuild models while preserving customizations and existing functionality
**Trigger**: Major schema changes, updated requirements, model restructuring needs

When regenerating models:

1. **Identify Custom Modifications**
   - Detect custom fields added beyond original requirements
   - Identify custom validation rules or business logic
   - Document custom relationships or indexes
   - Note any configuration changes from standard patterns

2. **Analyze Dependencies**
   - Find all imports of the model across codebase
   - Identify dependent services, APIs, and components
   - Check for existing migration files or database scripts
   - Document potential breaking changes

### Safe Regeneration Process
Regenerate while preserving customizations:

```python
class RegeneratedModel(Base):
    # New structure based on updated requirements
    {updated_fields_from_new_requirements}
    
    # Preserved custom modifications
    {preserved_custom_fields}
    
    # Updated relationships
    {updated_and_preserved_relationships}
```

Create migration strategy:
- Database migration scripts for schema changes
- Data migration procedures for existing data
- Rollback procedures for safe deployment
- Testing validation for migration success

---

## VALIDATION AND QUALITY ASSURANCE

### Automated Validation Checklist
Verify generated models meet all requirements:

✅ **File Structure**
- Model file created in backend/models/{ModelName}.py
- Proper imports from models.base import Base
- SQLAlchemy model inherits from Base with __tablename__

✅ **Model Definition**
- __table_args__ = {'extend_existing': True} included
- Primary key: id = Column(Integer, primary_key=True, index=True)
- Timestamp fields with proper func.now() defaults
- All metadata fields properly mapped to SQLAlchemy types

✅ **Pydantic Schemas**
- Complete schema set: Base, Create, Update, Response
- Response schema includes from_attributes = True
- Field validation with proper constraints and descriptions
- Optional fields properly typed for Update schema

✅ **Integration Requirements**
- Model imported in TableManager._import_all_models()
- No duplicate index definitions
- All imports reference existing files
- Proper nullable constraints based on requirements

✅ **Business Rules**
- All validation rules from metadata implemented
- Calculation logic for computed fields
- Cross-field validation for complex rules
- User-friendly error messages

### Testing Implementation
Generate comprehensive test coverage:

1. **Model Creation Tests**
   - Test table creation through TableManager
   - Verify all fields and constraints
   - Test relationship creation and foreign keys

2. **Pydantic Validation Tests**
   - Test all schemas with valid data
   - Test validation with invalid data
   - Test edge cases and boundary conditions

3. **Business Rules Tests**
   - Test all calculation logic
   - Test cross-field validation rules
   - Test constraint enforcement

4. **Integration Tests**
   - Test model registration with TableManager
   - Test database operations (CRUD)
   - Test API integration with schemas

---

## ERROR HANDLING AND TROUBLESHOOTING

### Common Issues and Solutions

**SQLAlchemy Issues:**
- **Duplicate Index Error**: Remove redundant index=True declarations
- **Foreign Key Issues**: Verify referenced tables exist and are imported
- **Column Type Mismatches**: Ensure proper SQLAlchemy types for data requirements

**Pydantic Issues:**
- **Validation Errors**: Align Field constraints with SQLAlchemy definitions
- **Serialization Issues**: Include from_attributes = True in Response Config
- **Optional Field Issues**: Use Optional[Type] for nullable columns

**TableManager Issues:**
- **Import Errors**: Verify model import in _import_all_models()
- **Circular Import Issues**: Manage import order for relationships
- **Table Creation Failures**: Ensure extend_existing: True in table_args

### Quality Standards
Ensure all generated code meets framework standards:
- Complete implementations with no placeholder comments
- Proper error handling and user-friendly messages
- Comprehensive documentation and type hints
- Framework consistency with established patterns
- Security considerations for data validation
- Performance optimization for database operations

## Success Criteria
- Models successfully integrate with existing TableManager system
- All metadata requirements accurately implemented in SQLAlchemy
- Complete Pydantic schema set supports full CRUD operations
- Business rules and validation logic properly enforced
- Database relationships correctly established
- Comprehensive testing validates all functionality
- Code follows established project patterns and conventions

Generate complete, production-ready SQLAlchemy models with comprehensive Pydantic schemas that seamlessly integrate with the existing FastAPI + SQLAlchemy + react-use-wizard technology stack while maintaining full backward compatibility and framework consistency.
