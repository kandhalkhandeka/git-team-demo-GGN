# RULES ENGINE - Business Logic Service Generation

## Command Patterns
When user inputs any of:
- `create rules` 
- `generate business rules`
- `create FastAPI services`
- `build validation services`
- `extend existing rules`
- `update validation`

## Purpose
Generate comprehensive FastAPI service layers with business rule validation, DataAccessController integration, and complete CRUD operations based on widget requirements and metadata specifications.

## Prerequisites
- Domain models have been created with proper Pydantic schemas
- Widget requirements and business rules are defined
- Existing service patterns have been analyzed
- DataAccessController integration requirements understood

## Mode Detection
Automatically detect execution mode based on context:
- **create-rules**: Fresh service generation from widget requirements
- **extend-rules**: Add new business rules to existing services
- **update-validation**: Update business rules and validation logic

---

## DISCOVERY AND ANALYSIS - Service Architecture Analysis

### Service Architecture Discovery - Pattern Recognition
Analyze existing codebase to understand established patterns:

1. **Examine Service Documentation**
   - Check `README.md` and `docs/README.md` (if available) for business domain context, API patterns, and service design principles
   - Review PDM Context: Analyze `./metadata/pdm.json` for business rule entities, validation requirements, and data constraints
   - Search workspace for service architecture documentation files
   - Extract service creation patterns and API design conventions
   - Identify integration requirements and established guidelines
   - Document FastAPI service development standards

2. **Analyze Existing Services**
   - Review all files in backend/services/ directory
   - Extract service structure patterns and endpoint conventions
   - Identify DataAccessController integration patterns
   - Document error handling and logging conventions
   - Collect best practices from existing implementations

3. **Study Integration Requirements**
   - Locate and analyze main.py service registration patterns
   - Understand service import and routing systems
   - Document business rule implementation patterns
   - Extract validation and transformation requirements

### Consolidation and Planning - Service Implementation Strategy
Consolidate all discovered patterns into a comprehensive service generation strategy that maintains consistency with existing codebase while implementing new business rules.

---

## SERVICE GENERATION EXECUTION - FastAPI Service Development

## CREATE-RULES MODE

### Purpose
Generate complete FastAPI service layer with comprehensive business rule validation, DataAccessController integration, and testing framework.

### FastAPI Service Creation - Complete Service Architecture

### Prerequisites
- Domain models have been created with proper Pydantic schemas
- Widget requirements and business rules are defined
- Existing service patterns have been analyzed

### Service Architecture Analysis - Pattern Discovery
Before generating services, analyze existing patterns:

#### 1. Discover Service Conventions
Analyze existing backend/services/ directory structure, `.github/chatmodes/Govern.chatmode.md` guidelines, and main.py registration patterns to understand:
- Service creation patterns and API design conventions
- DataAccessController integration requirements with .model_dump() usage
- Error handling patterns and logging conventions
- Service registration and import patterns in main.py

#### 2. Extract Business Requirements
Parse widget requirements to identify:
- Business rules and validation logic requirements
- Required service operations and CRUD endpoints
- Data validation and transformation needs
- Domain models and relationship requirements
- Integration points with existing services

#### 3. Plan Implementation Strategy
Create comprehensive service implementation plan including:
- FastAPI router structure with proper prefix and tags
- Complete endpoint architecture for all business operations
- DataAccessController integration strategy with proper error handling
- Business rule validation and data transformation logic
- Testing strategy following established `.github/chatmodes/Govern.chatmode.md` guidelines

### Service Generation Framework - Complete API Development

#### Core Service Structure - FastAPI Router Implementation
Generate complete FastAPI service following discovered patterns:

```python
from fastapi import APIRouter, HTTPException, Depends, status
from sqlalchemy.orm import Session
from controllers.DataAccessController import data_controller
from models.{ModelName} import {ModelName}, {ModelName}Create, {ModelName}Update, {ModelName}Response
from database import get_db
from typing import List
import logging

# Configure logging
logger = logging.getLogger(__name__)

# Create router with proper prefix and tags
router = APIRouter(
    prefix="/api/{servicename}",
    tags=["{servicename}"]
)
```

#### CRUD Operations Implementation - Complete Endpoint Generation
Generate complete CRUD endpoints with business rule validation:

**Create Endpoint with Validation - Data Creation Logic:**
```python
@router.post("/", response_model={ModelName}Response, status_code=status.HTTP_201_CREATED)
async def create_{entity}(
    item_data: {ModelName}Create,
    db: Session = Depends(get_db)
):
    """Create new {entity} with comprehensive business rule validation"""
    try:
        # Convert Pydantic model to dictionary (Required for DataAccessController)
        item_dict = item_data.model_dump()
        
        # Business rule validation based on widget requirements
        validate_{entity}_business_rules(item_dict)
        
        # Create through DataAccessController
        created_item = await data_controller.create({ModelName}, item_dict)
        logger.info(f"Created {entity} with ID: {created_item.id}")
        return created_item
        
    except ValueError as e:
        logger.error(f"Validation error creating {entity}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Validation error: {str(e)}"
        )
    except Exception as e:
        logger.error(f"Error creating {entity}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )
```

**Read Operations with Error Handling - Data Retrieval Logic:**
```python
@router.get("/{item_id}", response_model={ModelName}Response)
async def get_{entity}(item_id: int, db: Session = Depends(get_db)):
    """Retrieve {entity} by ID with proper error handling"""
    try:
        item = await data_controller.get_by_id({ModelName}, item_id)
        if not item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} with ID {item_id} not found"
            )
        return item
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error retrieving {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )

@router.get("/", response_model=List[{ModelName}Response])
async def list_{entities}(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    """List all {entities} with pagination"""
    try:
        items = await data_controller.get_all({ModelName}, skip=skip, limit=limit)
        return items
    except Exception as e:
        logger.error(f"Error listing {entities}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )
```

**Update Operations with Business Rule Validation - Data Modification Logic:**
```python
@router.put("/{item_id}", response_model={ModelName}Response)
async def update_{entity}(
    item_id: int,
    item_data: {ModelName}Update,
    db: Session = Depends(get_db)
):
    """Update {entity} with business rule validation"""
    try:
        # Verify item exists
        existing_item = await data_controller.get_by_id({ModelName}, item_id)
        if not existing_item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} with ID {item_id} not found"
            )
        
        # Convert to dictionary and filter None values
        update_dict = {k: v for k, v in item_data.model_dump().items() if v is not None}
        
        # Business rule validation for updates
        validate_{entity}_update_rules(update_dict, existing_item)
        
        # Update through DataAccessController
        updated_item = await data_controller.update({ModelName}, item_id, update_dict)
        logger.info(f"Updated {entity} with ID: {item_id}")
        return updated_item
        
    except HTTPException:
        raise
    except ValueError as e:
        logger.error(f"Validation error updating {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Validation error: {str(e)}"
        )
    except Exception as e:
        logger.error(f"Error updating {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )
```

**Delete Operations - Data Removal Logic:**
```python
@router.delete("/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_{entity}(item_id: int, db: Session = Depends(get_db)):
    """Delete {entity} by ID"""
    try:
        existing_item = await data_controller.get_by_id({ModelName}, item_id)
        if not existing_item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} with ID {item_id} not found"
            )
        
        await data_controller.delete({ModelName}, item_id)
        logger.info(f"Deleted {entity} with ID: {item_id}")
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error deleting {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )
```

### Business Rule Implementation - Validation Logic Framework
Transform widget requirements into comprehensive validation logic:

#### Data Validation Functions - Comprehensive Rule Engine
Generate validation functions based on widget business rules:

```python
def validate_{entity}_business_rules(data: dict) -> dict:
    """Comprehensive business rule validation for {entity}"""
    
    # Data type validation
    for field, value in data.items():
        if field in required_integer_fields and not isinstance(value, int):
            raise ValueError(f"{field} must be an integer")
        if field in required_string_fields and not isinstance(value, str):
            raise ValueError(f"{field} must be a string")
        if field in required_boolean_fields and not isinstance(value, bool):
            raise ValueError(f"{field} must be a boolean")
    
    # Field constraint validation
    for field, constraints in field_constraints.items():
        if field in data:
            if 'max_length' in constraints and len(str(data[field])) > constraints['max_length']:
                raise ValueError(f"{field} exceeds maximum length of {constraints['max_length']}")
            if 'min_value' in constraints and data[field] < constraints['min_value']:
                raise ValueError(f"{field} must be at least {constraints['min_value']}")
            if 'max_value' in constraints and data[field] > constraints['max_value']:
                raise ValueError(f"{field} cannot exceed {constraints['max_value']}")
    
    # Cross-field business rule validation
    if 'total_amount' in data and 'item_count' in data:
        if data['total_amount'] > 0 and data['item_count'] == 0:
            raise ValueError("Total amount cannot be greater than 0 when item count is 0")
    
    # Enum/constrained field validation
    for field, allowed_values in enum_fields.items():
        if field in data and data[field] not in allowed_values:
            raise ValueError(f"{field} must be one of: {', '.join(allowed_values)}")
    
    return data

def validate_{entity}_update_rules(data: dict, existing_item) -> dict:
    """Business rule validation for {entity} updates"""
    
    # Preserve critical relationships during updates
    if 'status' in data and existing_item.status == 'completed':
        raise ValueError("Cannot modify completed {entity}")
    
    # Update-specific validation logic
    if 'amount' in data and data['amount'] != existing_item.amount:
        if existing_item.related_transactions:
            raise ValueError("Cannot change amount when related transactions exist")
    
    return data
```

#### Calculation Engine Integration - Computed Field Logic
Implement calculation logic for computed fields:

```python
def calculate_{entity}_totals(data: dict) -> dict:
    """Calculate computed fields for {entity}"""
    
    # Automatic calculations based on widget requirements
    if 'quantity' in data and 'unit_price' in data:
        data['total_amount'] = data['quantity'] * data['unit_price']
    
    if 'base_amount' in data and 'tax_rate' in data:
        data['tax_amount'] = data['base_amount'] * (data['tax_rate'] / 100)
        data['final_amount'] = data['base_amount'] + data['tax_amount']
    
    return data
```

### Service Registration - Backend Integration
Register service in main.py following established patterns:

```python
# Service registration in main.py
from services.{ServiceName} import router as {servicename}_router
app.include_router({servicename}_router)
```

---

## EXTEND-RULES MODE

### Purpose
Enhance existing FastAPI services to support new widget requirements, business rules, and validation logic without disrupting existing functionality.

### Existing Service Analysis - Architecture Assessment

### Prerequisites
- Existing service structure has been analyzed
- New widget requirements have been identified
- Impact assessment has been completed

### Extension Strategy Framework - Safe Enhancement Implementation

#### Existing Service Analysis - Component Assessment
Before implementing extensions, analyze current service structure:
1. **Endpoint Assessment**: Review current endpoint structure and operations
2. **Business Logic Review**: Analyze existing business rule implementations
3. **Error Handling Patterns**: Document current error handling and logging
4. **DataAccessController Integration**: Verify existing integration points
5. **Testing Coverage**: Assess current test coverage and patterns

#### Safe Extension Implementation - Non-Breaking Enhancement

**Add New Endpoints - API Expansion:**
```python
# New endpoints following existing patterns
@router.post("/{entity}/batch", response_model=List[{ModelName}Response])
async def create_multiple_{entities}(
    items_data: List[{ModelName}Create],
    db: Session = Depends(get_db)
):
    """Batch create multiple {entities} with validation"""
    try:
        created_items = []
        for item_data in items_data:
            item_dict = item_data.model_dump()
            # Apply existing business rule validation
            validate_{entity}_business_rules(item_dict)
            created_item = await data_controller.create({ModelName}, item_dict)
            created_items.append(created_item)
        
        logger.info(f"Created {len(created_items)} {entities} in batch")
        return created_items
        
    except Exception as e:
        logger.error(f"Error in batch create {entities}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Batch creation failed"
        )

@router.post("/{entity}/validate", status_code=status.HTTP_200_OK)
async def validate_{entity}_data(
    item_data: {ModelName}Create,
    db: Session = Depends(get_db)
):
    """Validate {entity} data without creating record"""
    try:
        item_dict = item_data.model_dump()
        validate_{entity}_business_rules(item_dict)
        return {"status": "valid", "message": "Data validation successful"}
    except ValueError as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Validation error: {str(e)}"
        )
```

**Enhance Existing Business Rules - Validation Enhancement:**
```python
# Extend existing validation with new rules
def enhanced_{entity}_validation(data: dict) -> dict:
    """Enhanced validation including new business rules"""
    
    # Apply existing validation first
    data = validate_{entity}_business_rules(data)
    
    # Add new validation rules
    if 'new_field' in data:
        if data['new_field'] < 0:
            raise ValueError("New field cannot be negative")
    
    # New cross-field validation
    if 'field_a' in data and 'field_b' in data:
        if data['field_a'] > data['field_b']:
            raise ValueError("Field A cannot be greater than Field B")
    
    return data
```

**Backward Compatibility Preservation - Stability Assurance:**
- Maintain all existing endpoint signatures
- Preserve existing response formats
- Keep existing error handling patterns
- Maintain existing business rule behavior

---

## UPDATE-VALIDATION MODE

### Purpose
Update existing business rule validation logic, add new validation constraints, and enhance calculation algorithms without requiring complete service regeneration.

### Validation Enhancement Analysis - Business Rule Assessment

### Prerequisites
- Existing service structure has been analyzed
- New validation requirements have been identified from widget updates
- Validation update strategy has been planned to avoid breaking changes

### Validation Update Strategy Framework - Progressive Enhancement

#### Safe Validation Enhancement - Rule Improvement
Before updating validation, analyze current business rule structure:
1. **Validation Function Analysis**: Review current validation function implementations
2. **Business Rule Review**: Understand existing business rule patterns and constraints
3. **Calculation Logic Assessment**: Analyze current calculation and computation logic
4. **Error Handling Review**: Document existing error handling and response patterns
5. **Testing Coverage Analysis**: Assess current validation test coverage

#### Progressive Validation Updates - Enhanced Rule Implementation

**Enhanced Validation Functions - Advanced Rule Processing:**
```python
# Update existing validation with new rules while preserving existing logic
def enhanced_{entity}_validation(data: dict) -> dict:
    """Enhanced validation with new rules and improved constraints"""
    
    # Preserve existing validation first
    data = validate_existing_{entity}_rules(data)
    
    # Add new validation rules
    if 'new_constraint_field' in data:
        if not validate_new_constraint(data['new_constraint_field']):
            raise ValueError("New constraint field validation failed")
    
    # Enhanced cross-field validation
    if 'field_a' in data and 'field_b' in data and 'field_c' in data:
        if not validate_complex_business_rule(data['field_a'], data['field_b'], data['field_c']):
            raise ValueError("Complex business rule validation failed")
    
    # Updated calculation logic
    data = apply_enhanced_calculations(data)
    
    return data

def validate_new_constraint(value) -> bool:
    """New constraint validation logic"""
    # Implement new validation requirements
    if isinstance(value, str) and len(value) < 3:
        return False
    if isinstance(value, int) and value < 1:
        return False
    return True

def validate_complex_business_rule(field_a, field_b, field_c) -> bool:
    """Enhanced multi-field business rule validation"""
    # Implement complex business logic
    total = field_a + field_b
    if field_c > total * 1.5:
        return False
    return True

def apply_enhanced_calculations(data: dict) -> dict:
    """Apply updated calculation logic"""
    # Enhanced calculation algorithms
    if all(field in data for field in ['base_amount', 'tax_rate', 'discount_rate']):
        # New calculation including discount
        discounted_amount = data['base_amount'] * (1 - data['discount_rate'] / 100)
        data['tax_amount'] = discounted_amount * (data['tax_rate'] / 100)
        data['final_amount'] = discounted_amount + data['tax_amount']
    
    return data
```

**Backward Compatible Integration - Seamless Enhancement:**
```python
# Update existing service endpoints to use enhanced validation
@router.post("/", response_model={ModelName}Response, status_code=status.HTTP_201_CREATED)
async def create_{entity}_with_enhanced_validation(
    item_data: {ModelName}Create,
    db: Session = Depends(get_db)
):
    """Create {entity} with enhanced validation logic"""
    try:
        # Convert Pydantic model to dictionary
        item_dict = item_data.model_dump()
        
        # Apply enhanced validation (includes existing validation)
        validated_data = enhanced_{entity}_validation(item_dict)
        
        # Create through DataAccessController
        created_item = await data_controller.create({ModelName}, validated_data)
        logger.info(f"Created {entity} with enhanced validation, ID: {created_item.id}")
        return created_item
        
    except ValueError as e:
        logger.error(f"Enhanced validation error creating {entity}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Validation error: {str(e)}"
        )
    except Exception as e:
        logger.error(f"Error creating {entity} with enhanced validation: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )

@router.put("/{item_id}", response_model={ModelName}Response)
async def update_{entity}_with_enhanced_validation(
    item_id: int,
    item_data: {ModelName}Update,
    db: Session = Depends(get_db)
):
    """Update {entity} with enhanced validation logic"""
    try:
        # Verify item exists
        existing_item = await data_controller.get_by_id({ModelName}, item_id)
        if not existing_item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} with ID {item_id} not found"
            )
        
        # Convert to dictionary and filter None values
        update_dict = {k: v for k, v in item_data.model_dump().items() if v is not None}
        
        # Apply enhanced validation for updates
        validated_data = enhanced_{entity}_update_validation(update_dict, existing_item)
        
        # Update through DataAccessController
        updated_item = await data_controller.update({ModelName}, item_id, validated_data)
        logger.info(f"Updated {entity} with enhanced validation, ID: {item_id}")
        return updated_item
        
    except HTTPException:
        raise
    except ValueError as e:
        logger.error(f"Enhanced validation error updating {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Validation error: {str(e)}"
        )
    except Exception as e:
        logger.error(f"Error updating {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )
```

**Testing Enhanced Validation - Quality Assurance:**
```python
def test_enhanced_validation_rules():
    """Test new validation rules work correctly"""
    
    # Test new constraint validation
    test_data = {
        "existing_field": "valid_value",
        "new_constraint_field": "ab"  # Should fail new constraint
    }
    
    response = client.post("/api/{servicename}/", json=test_data)
    assert response.status_code == 400
    assert "new constraint field validation failed" in response.json()["detail"].lower()

def test_complex_business_rule_validation():
    """Test enhanced multi-field business rules"""
    
    test_data = {
        "field_a": 100,
        "field_b": 50,
        "field_c": 300  # Should fail: 300 > (100+50)*1.5 = 225
    }
    
    response = client.post("/api/{servicename}/", json=test_data)
    assert response.status_code == 400
    assert "complex business rule validation failed" in response.json()["detail"].lower()

def test_enhanced_calculation_logic():
    """Test updated calculation algorithms"""
    
    test_data = {
        "base_amount": 1000,
        "tax_rate": 10,
        "discount_rate": 15
    }
    
    response = client.post("/api/{servicename}/", json=test_data)
    assert response.status_code == 201
    
    data = response.json()
    # Verify enhanced calculations
    expected_discounted = 1000 * 0.85  # 15% discount
    expected_tax = expected_discounted * 0.10  # 10% tax on discounted amount
    expected_final = expected_discounted + expected_tax
    
    assert data["tax_amount"] == expected_tax
    assert data["final_amount"] == expected_final
```

---

## REGENERATE-RULES MODE

### Purpose
Completely regenerate FastAPI services from updated widget requirements while preserving valuable custom business logic and integrations.

### Command Context
When user inputs: `regenerate rules` or explicitly chooses complete business rule regeneration

### Prerequisites
- Existing service code has been analyzed
- Widget requirements have been updated
- User has explicitly chosen regeneration over extension or validation updates
- Current implementation has been backed up

### Regeneration Strategy Framework - Complete Service Regeneration

#### Preservation Analysis - Custom Logic Identification
Before regenerating, systematically identify and preserve:
1. **Custom Business Logic**: User-defined validation and calculation functions
2. **External Integrations**: Third-party API connections and service integrations
3. **Custom Endpoints**: Additional endpoints beyond standard CRUD operations
4. **Performance Optimizations**: Custom caching and optimization implementations
5. **Security Enhancements**: Additional security measures and middleware

#### Regeneration with Preservation - Enhanced Architecture Implementation

**Fresh Service Generation:**
- Generate new service structure based on updated widget requirements
- Implement enhanced business rules from updated specifications
- Create optimized endpoint architecture for current requirements
- Integrate preserved custom business logic where compatible
- Maintain existing external integrations and custom endpoints

**Custom Logic Integration - Business Logic Preservation:**
```python
# Preserve custom business logic in regenerated service
class Enhanced{ServiceName}:
    def __init__(self):
        # Preserve existing custom configurations
        self.custom_settings = preserve_custom_settings()
        
    async def create_with_custom_logic(self, data: dict):
        """Enhanced create with preserved custom logic"""
        # Apply new validation rules
        data = validate_new_business_rules(data)
        
        # Apply preserved custom logic
        data = apply_preserved_custom_logic(data)
        
        # Standard DataAccessController integration
        return await data_controller.create(ModelName, data)
```

---

## TESTING AND VALIDATION - Service Quality Assurance

## Comprehensive Testing Framework - FastAPI Testing Suite

### Test Suite Generation - Automated Service Testing
Create complete test coverage following `.github/chatmodes/Govern.chatmode.md` guidelines:

#### FastAPI TestClient Implementation - Service Testing Framework
```python
import pytest
from fastapi.testclient import TestClient
from main import app
from tests.test_utils import TestDatabase

client = TestClient(app)

class Test{ServiceName}:
    """Comprehensive test suite for {ServiceName} service"""
    
    def setup_method(self):
        """Setup test database and data"""
        self.test_db = TestDatabase()
        self.valid_data = {
            # Valid test data based on model requirements
        }
        self.invalid_data = {
            # Invalid test data for validation testing
        }
    
    def teardown_method(self):
        """Cleanup test data"""
        self.test_db.cleanup()
    
    def test_create_{entity}_success(self):
        """Test successful {entity} creation"""
        response = client.post("/api/{servicename}/", json=self.valid_data)
        assert response.status_code == 201
        data = response.json()
        assert data["id"] is not None
        assert all(field in data for field in required_fields)
    
    def test_create_{entity}_empty_request(self):
        """Test {entity} creation with empty request body"""
        response = client.post("/api/{servicename}/", json={})
        assert response.status_code == 422
    
    def test_business_rules_validation(self):
        """Test comprehensive business rule validation"""
        # Test each business rule with edge cases
        invalid_cases = [
            ({"field": "invalid_value"}, "field validation error"),
            ({"amount": -1}, "amount cannot be negative"),
            ({"status": "invalid"}, "status must be valid enum value")
        ]
        
        for invalid_data, expected_error in invalid_cases:
            response = client.post("/api/{servicename}/", json=invalid_data)
            assert response.status_code == 400
            assert expected_error.lower() in response.json()["detail"].lower()
```

#### Business Rule Testing - Validation Logic Testing
```python
def test_data_type_validation(self):
    """Test data type validation for all fields"""
    type_test_cases = [
        ("integer_field", "string_value", "must be an integer"),
        ("string_field", 123, "must be a string"),
        ("boolean_field", "not_boolean", "must be a boolean")
    ]
    
    for field, invalid_value, expected_error in type_test_cases:
        test_data = self.valid_data.copy()
        test_data[field] = invalid_value
        response = client.post("/api/{servicename}/", json=test_data)
        assert response.status_code == 400
        assert expected_error in response.json()["detail"]

def test_field_constraints(self):
    """Test field constraint validation"""
    constraint_test_cases = [
        ("name", "x" * 101, "exceeds maximum length"),
        ("age", -1, "must be at least"),
        ("percentage", 101, "cannot exceed")
    ]
    
    for field, invalid_value, expected_error in constraint_test_cases:
        test_data = self.valid_data.copy()
        test_data[field] = invalid_value
        response = client.post("/api/{servicename}/", json=test_data)
        assert response.status_code == 400
        assert expected_error in response.json()["detail"]

def test_cross_field_validation(self):
    """Test cross-field business rule validation"""
    # Test complex business rules involving multiple fields
    test_data = self.valid_data.copy()
    test_data["start_date"] = "2023-12-31"
    test_data["end_date"] = "2023-01-01"
    
    response = client.post("/api/{servicename}/", json=test_data)
    assert response.status_code == 400
    assert "end date cannot be before start date" in response.json()["detail"].lower()
```

---

## Quality Assurance Framework - Service Standards Compliance

### Validation Checklist - Service Requirements Verification
Verify generated services meet all requirements:

#### Service Structure Validation
- ✅ Service file created in backend/services/{ServiceName}.py
- ✅ APIRouter imported and instantiated with proper prefix and tags
- ✅ Service imported and registered in main.py with app.include_router()
- ✅ Standard CRUD endpoints (GET, POST, PUT, DELETE) defined with proper decorators

#### FastAPI Integration Validation
- ✅ All endpoints have response_model parameter specified
- ✅ HTTPException imported and used for error responses
- ✅ Proper status codes used for different operations
- ✅ Database dependency injection using Depends(get_db)

#### DataAccessController Integration Validation
- ✅ data_controller imported and used in all endpoints
- ✅ Pydantic models converted to dictionaries using .model_dump()
- ✅ Proper error handling around DataAccessController calls
- ✅ Correct model imports from model files

#### Business Rules Validation
- ✅ Data type validation implemented for all fields
- ✅ Field constraints validated before database operations
- ✅ Cross-field business rule validation with comprehensive edge cases
- ✅ Value validation for enums and constrained fields
- ✅ Calculation logic for computed fields

#### Error Handling and Logging Validation
- ✅ Comprehensive try-catch blocks around all operations
- ✅ Logger imported and configured at module level
- ✅ Backend logging with logger.error() for exception details
- ✅ Response format matches frontend expectations

#### Testing Validation
- ✅ Service tested with FastAPI TestClient including empty requests
- ✅ All CRUD operations tested with valid and invalid data
- ✅ Business rule validation tested with comprehensive edge cases
- ✅ Error scenarios properly tested and validated
- ✅ Performance testing for database operations

### Quality Standards - Framework Standards Compliance
Ensure all generated services meet framework standards:
- Complete implementations with no placeholder code
- Proper error handling with user-friendly messages
- Comprehensive logging for debugging and monitoring
- Framework consistency with established patterns
- Security considerations for data validation
- Performance optimization for database operations

<!-- COMPLETION VERIFICATION - Service-Specific Success Criteria -->
Before marking service generation complete, verify appropriate mode completion:

**For CREATE-RULES mode:**
☐ Complete FastAPI service structure with proper router configuration
☐ All widget business rules implemented with comprehensive validation
☐ DataAccessController integration follows established patterns with .model_dump()
☐ Standard CRUD operations implemented with proper error handling
☐ Business rule validation functions handle all widget requirements
☐ Service registration completed in main.py following patterns
☐ Comprehensive test suite covers all endpoints and business rules
☐ Error handling provides appropriate responses for validation failures
☐ Logging system properly configured for debugging and monitoring
☐ Performance considerations addressed for database operations

**For EXTEND-RULES mode:**
☐ New widget requirements successfully integrated into existing service structure
☐ Enhanced business rules work seamlessly with existing validation framework
☐ New endpoints follow existing patterns and maintain consistency
☐ Existing functionality continues to work without regression
☐ Backward compatibility maintained for all existing API consumers
☐ Test coverage extended to include new functionality and edge cases
☐ Error handling patterns consistent with existing implementation
☐ Performance characteristics maintained or improved with enhancements
☐ Documentation updated to reflect new functionality
☐ Integration testing validates enhanced service behavior

**For REGENERATE-RULES mode:**
☐ All widget business rules implemented in fresh service architecture
☐ Existing valuable custom logic successfully integrated into new structure
☐ Previous API functionality preserved where not superseded by new requirements
☐ Fresh endpoints follow updated widget specifications exactly
☐ DataAccessController integration patterns maintained throughout regeneration
☐ Custom integrations and optimizations preserved where beneficial
☐ Authentication and authorization compatibility maintained
☐ Performance characteristics maintained or improved compared to previous implementation
☐ Migration strategy planned for transitioning to regenerated service
☐ Comprehensive testing validates all regenerated functionality

**Universal verification for all modes:**
☐ Service response format consistent with FastAPI best practices
☐ All endpoints properly documented with request/response examples
☐ Database operations optimized with proper DataAccessController usage
☐ Error logging provides sufficient detail for debugging and monitoring
☐ Business rules engine accurately implements all widget specifications
☐ Security measures appropriate for service operations and data handling
☐ Code follows established project patterns and naming conventions
☐ Integration with existing backend infrastructure functions correctly
<!-- END VERIFICATION -->

Execute appropriate service generation mode based on user command context, ensuring comprehensive implementation of widget business rules while maintaining seamless integration with FastAPI + SQLAlchemy + DataAccessController technology stack.
