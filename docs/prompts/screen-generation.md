# SCREEN GENERATION - React Frontend Component Development

## Command Patterns
When user inputs any of:
- `build screens from metadata`
- `generate React screens` 
- `create screen components`
- `build frontend screens`
- `extend existing screens`
- `rebuild screens`
- `add fields to screens`

## Purpose
Generate comprehensive React screen components with backend integration, data persistence, and navigation configuration based on metadata field specifications, following established project patterns with complete full-stack implementation.

## Prerequisites
- Domain models have been created with proper Pydantic schemas
- Widget requirements and screen functionality are defined
- Existing screen patterns have been analyzed
- Metadata files analyzed and requirements extracted
- Project technology stack configured (React + FastAPI + SQLAlchemy + DataAccessController)

## Mode Detection
Automatically detect execution mode based on context:
- **build-screens**: Fresh screen generation from widget requirements
- **extend-screens**: Add new functionality to existing screens
- **rebuild-screens**: Rebuild screens while preserving custom logic
- **add-fields**: Add new fields to existing screens without rebuilding

---

## DISCOVERY AND ANALYSIS - Codebase Pattern Recognition

### Architecture Pattern Discovery
Analyze existing codebase to understand established patterns:

1. **Examine Frontend Conventions**
   - Check `README.md` and `docs/README.md` (if available) for UI/UX guidelines, component patterns, and screen design standards
   - Search workspace for React component documentation files
   - Extract screen creation patterns and naming conventions
   - Identify state management approaches and hooks usage patterns
   - Document API integration patterns and HTTP client usage
   - Analyze styling conventions and responsive design patterns

2. **Analyze Existing Screens**
   - Review all files in frontend/src/screens/ directory structure
   - Extract component structure patterns and established conventions
   - Identify navigation patterns and routing configurations
   - Document integration points with existing components
   - Collect best practices from existing implementations

3. **Study Backend Service Integration**
   - Locate and analyze existing FastAPI service patterns
   - Understand service registration in main.py patterns
   - Document API endpoint patterns and DataAccessController usage
   - Extract service-to-model integration requirements
   - Analyze navigation integration with left-navigation.json

4. **Process Widget Requirements**
   - Parse widget metadata field specifications and validation rules
   - Map widget field types to React component types
   - Extract screen functionality and user flow requirements
   - Identify data collection and persistence requirements
   - Document entity relationships and integration points

### Consolidation and Planning - Implementation Strategy
Create comprehensive implementation approach:
- Identify conflicts between existing patterns and new requirements
- Plan integration strategy that maintains consistency with established patterns
- Design backward-compatible extensions for existing screens
- Establish testing and validation approach for full-stack implementation

---

## SCREEN GENERATION EXECUTION - Mode-Based Implementation

## BUILD-SCREENS MODE

### Purpose
Generate complete full-stack screen implementation including React component, backend services, data persistence, and navigation integration.

### Command Context
When user inputs: `build screens` for fresh screen generation from widget requirements

### Prerequisites
- Domain models have been created with proper Pydantic schemas
- Widget requirements and screen functionality are defined
- Existing screen patterns have been analyzed

### Frontend Architecture Analysis
Before generating screens, analyze existing patterns:

#### 1. Discover Frontend Conventions
Analyze existing frontend/src/screens/ directory structure, `.github/chatmodes/Govern.chatmode.md` guidelines, and component patterns to understand:
- Component creation patterns and React conventions
- State management approaches and hooks usage
- API integration patterns and HTTP client usage
- Styling conventions and responsive design patterns

#### 2. Extract Screen Requirements
Parse widget requirements to identify:
- Screen functionality and user flow requirements
- Data collection and validation needs
- Entity relationships and data persistence requirements
- Navigation and routing requirements
- Integration points with existing components

#### 3. Plan Backend Integration
Create comprehensive implementation plan including:
- Backend service architecture with required endpoints
- API integration strategy using established patterns
- Data persistence strategy for all collected entities
- Navigation integration with left-navigation.json

### Full-Stack Implementation Framework - Complete Component Generation

#### Backend Service Generation - FastAPI Service Creation
Generate complete FastAPI service supporting screen functionality:

```python
from fastapi import APIRouter, HTTPException, Depends, status
from sqlalchemy.orm import Session
from controllers.DataAccessController import data_controller
from models.{ModelName} import {ModelName}, {ModelName}Create, {ModelName}Update, {ModelName}Response
from database import get_db
from typing import List, Dict, Any
import logging

# Configure logging
logger = logging.getLogger(__name__)

# Create router with proper prefix and tags
router = APIRouter(
    prefix="/api/{screenname}",
    tags=["{screenname}"]
)

@router.post("/", response_model={ModelName}Response, status_code=status.HTTP_201_CREATED)
async def create_{entity}(
    screen_data: {ModelName}Create,
    db: Session = Depends(get_db)
):
    """Create new {entity} with comprehensive validation and related data processing"""
    try:
        # Convert Pydantic model to dictionary (Required for DataAccessController)
        data_dict = screen_data.model_dump()
        
        # Validate screen data according to widget requirements
        validate_screen_data(data_dict)
        
        # Process and persist primary entity
        created_item = await data_controller.create({ModelName}, data_dict)
        
        # Process related entities collected on screen
        await process_related_entities(created_item.id, data_dict)
        
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

@router.get("/{item_id}", response_model={ModelName}Response)
async def get_{entity}(item_id: int, db: Session = Depends(get_db)):
    """Retrieve {entity} with related data for screen display"""
    try:
        item = await data_controller.get_by_id({ModelName}, item_id)
        if not item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} with ID {item_id} not found"
            )
        
        # Load related data for screen display
        item_data = await load_related_screen_data(item)
        return item_data
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error retrieving {entity} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error occurred"
        )

@router.put("/{item_id}", response_model={ModelName}Response)
async def update_{entity}(
    item_id: int,
    screen_data: {ModelName}Update,
    db: Session = Depends(get_db)
):
    """Update {entity} and related data from screen"""
    try:
        # Verify item exists
        existing_item = await data_controller.get_by_id({ModelName}, item_id)
        if not existing_item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} with ID {item_id} not found"
            )
        
        # Convert to dictionary and filter None values
        update_dict = {k: v for k, v in screen_data.model_dump().items() if v is not None}
        
        # Validate update data
        validate_screen_update_data(update_dict, existing_item)
        
        # Update primary entity
        updated_item = await data_controller.update({ModelName}, item_id, update_dict)
        
        # Update related entities
        await update_related_entities(item_id, update_dict)
        
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

async def process_related_entities(primary_id: int, data: Dict[str, Any]):
    """Process and persist all related entities collected on screen"""
    try:
        # Process each related entity type based on screen requirements
        if 'related_items' in data:
            for item_data in data['related_items']:
                item_data['primary_entity_id'] = primary_id
                await data_controller.create(RelatedModel, item_data)
        
        # Process additional relationships as needed
        if 'secondary_data' in data:
            secondary_data = data['secondary_data']
            secondary_data['primary_entity_id'] = primary_id
            await data_controller.create(SecondaryModel, secondary_data)
            
    except Exception as e:
        logger.error(f"Error processing related entities: {str(e)}")
        raise

def validate_screen_data(data: Dict[str, Any]) -> Dict[str, Any]:
    """Comprehensive validation for screen data"""
    
    # Data type validation
    for field, value in data.items():
        if field in required_integer_fields and not isinstance(value, int):
            raise ValueError(f"{field} must be an integer")
        if field in required_string_fields and not isinstance(value, str):
            raise ValueError(f"{field} must be a string")
        if field in date_fields and value:
            try:
                datetime.strptime(value, "%Y-%m-%d")
            except ValueError:
                raise ValueError(f"{field} must be a valid date in YYYY-MM-DD format")
    
    # Field constraint validation
    for field, constraints in field_constraints.items():
        if field in data and data[field]:
            if 'max_length' in constraints and len(str(data[field])) > constraints['max_length']:
                raise ValueError(f"{field} exceeds maximum length of {constraints['max_length']}")
            if 'required' in constraints and constraints['required'] and not data[field]:
                raise ValueError(f"{field} is required")
    
    # Dropdown/enum validation
    for field, allowed_values in dropdown_fields.items():
        if field in data and data[field] not in allowed_values:
            raise ValueError(f"{field} must be one of: {', '.join(allowed_values)}")
    
    return data
```

#### React Component Generation - Interactive Form Components
Generate complete React screen component with proper patterns:

```jsx
import React, { useState, useEffect } from 'react';
import { 
    Box, 
    Paper, 
    Typography, 
    TextField, 
    Button, 
    Select, 
    MenuItem, 
    FormControl, 
    InputLabel, 
    Alert,
    CircularProgress,
    Grid
} from '@mui/material';
import { useNavigate, useParams } from 'react-router-dom';
import './{ComponentName}.css';

const {ComponentName} = () => {
    // State management for form data
    const [formData, setFormData] = useState({
        // Initialize based on widget requirements
    });
    
    // State management for UI
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    
    // Navigation and routing
    const navigate = useNavigate();
    const { id } = useParams();
    const isEditMode = Boolean(id);
    
    // Load data for edit mode
    useEffect(() => {
        if (isEditMode) {
            loadEntityData();
        }
    }, [id]);
    
    const loadEntityData = async () => {
        setLoading(true);
        setError('');
        
        try {
            const response = await fetch(`/api/{servicename}/${id}`);
            
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.detail || 'Failed to load data');
            }
            
            const data = await response.json();
            setFormData(data);
            
        } catch (err) {
            console.error('Error loading entity data:', err);
            setError(`Failed to load data: ${err.message}`);
        } finally {
            setLoading(false);
        }
    };
    
    const handleInputChange = (field) => (event) => {
        const value = event.target.value;
        setFormData(prev => ({
            ...prev,
            [field]: value
        }));
        
        // Clear error when user starts typing
        if (error) setError('');
    };
    
    const handleSubmit = async (event) => {
        event.preventDefault();
        setLoading(true);
        setError('');
        setSuccess('');
        
        try {
            // Validate form data before submission
            validateFormData();
            
            const url = isEditMode ? `/api/{servicename}/${id}` : '/api/{servicename}/';
            const method = isEditMode ? 'PUT' : 'POST';
            
            const response = await fetch(url, {
                method,
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(formData)
            });
            
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.detail || `Failed to ${isEditMode ? 'update' : 'create'} entity`);
            }
            
            const result = await response.json();
            setSuccess(`Entity ${isEditMode ? 'updated' : 'created'} successfully!`);
            
            // Navigate after successful submission
            setTimeout(() => {
                navigate('/entity-list'); // Adjust navigation target as needed
            }, 2000);
            
        } catch (err) {
            console.error('Error submitting form:', err);
            setError(`Submission failed: ${err.message}`);
        } finally {
            setLoading(false);
        }
    };
    
    const validateFormData = () => {
        // Client-side validation based on widget requirements
        const requiredFields = ['name', 'category']; // Adjust based on requirements
        
        for (const field of requiredFields) {
            if (!formData[field]?.trim()) {
                throw new Error(`${field} is required`);
            }
        }
        
        // Additional validation logic based on widget requirements
        if (formData.amount && formData.amount < 0) {
            throw new Error('Amount cannot be negative');
        }
    };
    
    const handleCancel = () => {
        navigate(-1); // Go back to previous page
    };
    
    if (loading && isEditMode && !formData.id) {
        return (
            <Box display="flex" justifyContent="center" alignItems="center" minHeight="200px">
                <CircularProgress />
            </Box>
        );
    }
    
    return (
        <Box className="{componentname}-container">
            <Paper elevation={3} className="{componentname}-paper">
                <Typography variant="h4" component="h1" gutterBottom>
                    {isEditMode ? 'Edit' : 'Create'} {EntityName}
                </Typography>
                
                {error && (
                    <Alert severity="error" sx={{ mb: 2 }}>
                        <Typography variant="body2">
                            <strong>Error:</strong> {error}
                        </Typography>
                    </Alert>
                )}
                
                {success && (
                    <Alert severity="success" sx={{ mb: 2 }}>
                        {success}
                    </Alert>
                )}
                
                <form onSubmit={handleSubmit}>
                    <Grid container spacing={3}>
                        {/* Generate form fields based on widget requirements */}
                        <Grid item xs={12} md={6}>
                            <TextField
                                fullWidth
                                label="Name"
                                value={formData.name || ''}
                                onChange={handleInputChange('name')}
                                required
                                error={!formData.name?.trim() && error}
                                helperText={!formData.name?.trim() && error ? 'Name is required' : ''}
                            />
                        </Grid>
                        
                        <Grid item xs={12} md={6}>
                            <FormControl fullWidth>
                                <InputLabel>Category</InputLabel>
                                <Select
                                    value={formData.category || ''}
                                    onChange={handleInputChange('category')}
                                    label="Category"
                                    required
                                >
                                    <MenuItem value="option1">Option 1</MenuItem>
                                    <MenuItem value="option2">Option 2</MenuItem>
                                    {/* Generate options based on widget requirements */}
                                </Select>
                            </FormControl>
                        </Grid>
                        
                        {/* Additional form fields based on widget requirements */}
                        <Grid item xs={12} md={6}>
                            <TextField
                                fullWidth
                                label="Amount"
                                type="number"
                                value={formData.amount || ''}
                                onChange={handleInputChange('amount')}
                                inputProps={{ min: 0, step: 0.01 }}
                            />
                        </Grid>
                        
                        <Grid item xs={12}>
                            <TextField
                                fullWidth
                                label="Description"
                                multiline
                                rows={4}
                                value={formData.description || ''}
                                onChange={handleInputChange('description')}
                            />
                        </Grid>
                    </Grid>
                    
                    <Box sx={{ mt: 3, display: 'flex', gap: 2, justifyContent: 'flex-end' }}>
                        <Button
                            variant="outlined"
                            onClick={handleCancel}
                            disabled={loading}
                        >
                            Cancel
                        </Button>
                        <Button
                            type="submit"
                            variant="contained"
                            disabled={loading}
                            startIcon={loading ? <CircularProgress size={20} /> : null}
                        >
                            {loading ? 'Saving...' : (isEditMode ? 'Update' : 'Create')}
                        </Button>
                    </Box>
                </form>
            </Paper>
        </Box>
    );
};

export default {ComponentName};
```

#### Component Styling - Responsive Design Implementation
Generate responsive CSS with accessibility considerations:

```css
.{componentname}-container {
    padding: 24px;
    max-width: 1200px;
    margin: 0 auto;
}

.{componentname}-paper {
    padding: 32px;
    margin-bottom: 24px;
}

.{componentname}-form-section {
    margin-bottom: 24px;
}

.{componentname}-form-section:last-child {
    margin-bottom: 0;
}

/* Responsive design */
@media (max-width: 768px) {
    .{componentname}-container {
        padding: 16px;
    }
    
    .{componentname}-paper {
        padding: 16px;
    }
}

/* Accessibility enhancements */
.{componentname}-container input:focus,
.{componentname}-container select:focus,
.{componentname}-container textarea:focus {
    outline: 2px solid #1976d2;
    outline-offset: 2px;
}

/* Loading state styling */
.{componentname}-loading {
    opacity: 0.6;
    pointer-events: none;
}

/* Error styling */
.{componentname}-error {
    color: #d32f2f;
    margin-top: 8px;
}
```

### Navigation Integration - Screen Registration
Register screen in left-navigation.json following established patterns:

```json
{
    "path": "/{screenname}",
    "name": "{ScreenName}",
    "component": "{ComponentName}",
    "showInNavigation": true,
    "icon": "AddIcon",
    "order": 10
}
```

### Service Registration - Backend Integration
Register service in main.py following established patterns:

```python
# Service registration in main.py
from services.{ScreenName}Service import router as {screenname}_router
app.include_router({screenname}_router)
```

---

---

## EXTEND-SCREENS MODE

### Purpose
Enhance existing React screens and backend services to support new widget requirements and functionality without disrupting existing behavior.

### Command Context
When user inputs: `extend screens` for enhancing existing screens with new functionality

### Prerequisites
- Existing screen structure has been analyzed
- New widget requirements have been identified
- Impact assessment has been completed

### Extension Strategy Framework - Safe Enhancement Implementation

#### Existing Screen Analysis - Component Assessment
Before implementing extensions, analyze current screen structure:
1. **Component Assessment**: Review current component structure and state management
2. **API Integration Review**: Analyze existing API integration patterns
3. **Backend Service Review**: Document current service endpoints and functionality
4. **Navigation Impact**: Assess navigation and routing changes needed
5. **Data Flow Analysis**: Understand current data persistence and relationships

#### Safe Extension Implementation - Backward Compatible Enhancement

**Enhance Frontend Component:**
```jsx
// Add new form sections while preserving existing functionality
const Enhanced{ComponentName} = () => {
    // Preserve existing state
    const [existingFormData, setExistingFormData] = useState({...});
    
    // Add new state for enhanced functionality
    const [newFormSection, setNewFormSection] = useState({});
    const [additionalData, setAdditionalData] = useState([]);
    
    // Preserve existing form handlers
    const handleExistingSubmit = async (event) => {
        // Existing submission logic preserved
        const existingData = await submitExistingData();
        
        // Add new data processing
        await submitAdditionalData();
    };
    
    // Add new form sections
    const renderNewFormSection = () => (
        <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
                <TextField
                    fullWidth
                    label="New Field"
                    value={newFormSection.newField || ''}
                    onChange={handleNewFieldChange('newField')}
                />
            </Grid>
            {/* Additional new fields */}
        </Grid>
    );
    
    return (
        <Box>
            {/* Preserve existing form sections */}
            {renderExistingFormSections()}
            
            {/* Add new form sections */}
            <Typography variant="h6" gutterBottom sx={{ mt: 3 }}>
                Additional Information
            </Typography>
            {renderNewFormSection()}
        </Box>
    );
};
```

**Extend Backend Service:**
```python
# Add new endpoints while preserving existing functionality
@router.post("/{entity}/additional-data", response_model=dict)
async def save_additional_data(
    entity_id: int,
    additional_data: AdditionalDataCreate,
    db: Session = Depends(get_db)
):
    """Save additional data collected from enhanced screen"""
    try:
        # Verify parent entity exists
        parent_entity = await data_controller.get_by_id(MainModel, entity_id)
        if not parent_entity:
            raise HTTPException(status_code=404, detail="Parent entity not found")
        
        # Process additional data
        data_dict = additional_data.model_dump()
        data_dict['parent_entity_id'] = entity_id
        
        result = await data_controller.create(AdditionalDataModel, data_dict)
        return {"status": "success", "id": result.id}
        
    except Exception as e:
        logger.error(f"Error saving additional data: {str(e)}")
        raise HTTPException(status_code=500, detail="Failed to save additional data")
```

**Backward Compatibility Preservation:**
- Maintain all existing component props and state structure
- Preserve existing API endpoint signatures and responses
- Keep existing navigation patterns and routing
- Maintain existing CSS classes and styling conventions

---

---

## REBUILD-SCREENS MODE

### Purpose
Completely rebuild React screens and backend services from updated widget requirements while preserving valuable custom logic and integrations.

### Command Context
When user inputs: `rebuild screens` or explicitly chooses screen rebuilding approach

### Prerequisites
- Existing screen code has been analyzed
- Widget requirements have been updated
- User has explicitly chosen rebuilding over extension
- Current implementation has been backed up

### Rebuilding Strategy Framework - Complete Component Regeneration

#### Preservation Analysis - Custom Logic Identification
Before regenerating, systematically identify and preserve:
1. **Custom Component Logic**: User-defined validation and interaction patterns
2. **Styling Customizations**: Custom CSS and theme modifications
3. **API Integration Patterns**: Custom HTTP client configurations and error handling
4. **Navigation Customizations**: Custom routing and navigation behavior
5. **Performance Optimizations**: Custom caching and optimization implementations

#### Rebuilding with Preservation - Fresh Architecture Implementation

**Fresh Component Generation:**
- Generate new component structure based on updated widget requirements
- Implement enhanced form patterns from updated specifications
- Create optimized state management for current requirements
- Integrate preserved custom logic where compatible
- Maintain existing navigation and routing patterns

**Custom Logic Integration:**
```jsx
// Preserve custom logic in rebuilt component
const Enhanced{ComponentName} = () => {
    // Preserve existing custom configurations
    const [customSettings, setCustomSettings] = useState(preserveCustomSettings());
    
    // Apply new form structure with preserved logic
    const handleEnhancedSubmit = async (data) => {
        // Apply new validation rules
        const validatedData = validateNewRequirements(data);
        
        // Apply preserved custom logic
        const processedData = applyPreservedCustomLogic(validatedData);
        
        // Submit with enhanced API integration
        return await submitWithNewEndpoints(processedData);
    };
    
    return (
        <Box>
            {/* New form structure based on updated requirements */}
            {renderEnhancedFormSections()}
            
            {/* Preserved custom components */}
            {renderPreservedCustomSections()}
        </Box>
    );
};
```

---

---

## ADD-FIELDS MODE

### Purpose
Add new form fields and corresponding backend support to existing React screens and FastAPI services without disrupting existing functionality or requiring full screen rebuilds.

### Command Context
When user inputs: `add fields` for adding new fields to existing screens without rebuilding

### Prerequisites
- Existing screen structure has been analyzed
- New field requirements have been identified from widget updates
- Field addition strategy has been planned to avoid conflicts

### Field Addition Strategy Framework - Progressive Enhancement

#### Safe Field Integration - Form Structure Analysis
Before adding fields, analyze current screen structure:
1. **Form Structure Analysis**: Review current form layout and field organization
2. **State Management Review**: Understand how form state is currently managed
3. **API Contract Analysis**: Review existing API endpoints and data structures
4. **Validation Pattern Review**: Understand current validation implementation
5. **Backend Model Analysis**: Review existing data models and database schema

#### Progressive Field Addition - Non-Breaking Enhancement

**Frontend Field Integration:**
```jsx
// Add new fields while preserving existing form structure
const Enhanced{ComponentName} = () => {
    // Preserve existing form state
    const [existingFormData, setExistingFormData] = useState({
        // existing fields preserved
    });
    
    // Add new field state
    const [newFields, setNewFields] = useState({
        newField1: '',
        newField2: '',
        newDropdownField: ''
    });
    
    // Preserve existing form handlers
    const handleExistingFieldChange = (field) => (event) => {
        setExistingFormData(prev => ({
            ...prev,
            [field]: event.target.value
        }));
    };
    
    // Add new field handlers
    const handleNewFieldChange = (field) => (event) => {
        setNewFields(prev => ({
            ...prev,
            [field]: event.target.value
        }));
    };
    
    // Enhanced submit handler combining old and new data
    const handleSubmit = async (event) => {
        event.preventDefault();
        
        const combinedData = {
            ...existingFormData,
            ...newFields
        };
        
        // Submit with enhanced data structure
        await submitEnhancedForm(combinedData);
    };
    
    // Add new form sections while preserving existing layout
    const renderNewFields = () => (
        <Grid container spacing={3}>
            <Grid item xs={12} md={6}>
                <TextField
                    fullWidth
                    label="New Field 1"
                    value={newFields.newField1}
                    onChange={handleNewFieldChange('newField1')}
                    required
                />
            </Grid>
            <Grid item xs={12} md={6}>
                <FormControl fullWidth>
                    <InputLabel>New Dropdown Field</InputLabel>
                    <Select
                        value={newFields.newDropdownField}
                        onChange={handleNewFieldChange('newDropdownField')}
                    >
                        <MenuItem value="option1">Option 1</MenuItem>
                        <MenuItem value="option2">Option 2</MenuItem>
                    </Select>
                </FormControl>
            </Grid>
        </Grid>
    );
    
    return (
        <Box className="{componentname}-container">
            <Paper elevation={3} className="{componentname}-paper">
                <form onSubmit={handleSubmit}>
                    {/* Existing form sections preserved */}
                    <div className="{componentname}-existing-section">
                        {/* existing form fields remain unchanged */}
                    </div>
                    
                    {/* New fields section */}
                    <div className="{componentname}-new-fields-section">
                        <Typography variant="h6" gutterBottom>
                            Additional Information
                        </Typography>
                        {renderNewFields()}
                    </div>
                    
                    {/* Enhanced submit button */}
                    <Button
                        type="submit"
                        variant="contained"
                        color="primary"
                        disabled={loading}
                    >
                        {loading ? 'Saving...' : (isEditMode ? 'Update' : 'Create')}
                    </Button>
                </form>
            </Paper>
        </Box>
    );
};
```

**Backend Model Extension - Schema Enhancement:**
```python
# Extend existing Pydantic models with new fields
class Enhanced{ModelName}Create({ModelName}Create):
    """Extended create model with new fields"""
    
    # New fields as optional to maintain backward compatibility
    new_field1: Optional[str] = None
    new_field2: Optional[str] = None
    new_dropdown_field: Optional[str] = None
    
    @validator('new_dropdown_field')
    def validate_new_dropdown(cls, v):
        if v is not None and v not in ['option1', 'option2']:
            raise ValueError('new_dropdown_field must be option1 or option2')
        return v

class Enhanced{ModelName}Update({ModelName}Update):
    """Extended update model with new fields"""
    
    # Add new optional fields for updates
    new_field1: Optional[str] = None
    new_field2: Optional[str] = None
    new_dropdown_field: Optional[str] = None

# Extend SQLAlchemy model with new columns
# Note: This requires database migration
class Enhanced{ModelName}({ModelName}):
    """Extended model with new database columns"""
    
    new_field1 = Column(String(255), nullable=True)
    new_field2 = Column(String(500), nullable=True) 
    new_dropdown_field = Column(String(50), nullable=True)
```

**Backend Service Extension - API Enhancement:**
```python
# Extend existing service endpoints to handle new fields
@router.put("/{item_id}/add-fields", response_model={ModelName}Response)
async def add_fields_to_item(
    item_id: int,
    field_data: Enhanced{ModelName}Update,
    db: Session = Depends(get_db)
):
    """Add new fields to existing item without affecting existing data"""
    try:
        # Verify item exists
        existing_item = await data_controller.get_by_id({ModelName}, item_id)
        if not existing_item:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"{ModelName} not found"
            )
        
        # Extract only new fields, filtering None values
        new_field_data = {
            k: v for k, v in field_data.model_dump().items() 
            if v is not None and k.startswith('new_')
        }
        
        # Update only new fields
        updated_item = await data_controller.update({ModelName}, item_id, new_field_data)
        
        logger.info(f"Added fields to {ModelName} with ID: {item_id}")
        return updated_item
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error adding fields to {ModelName} {item_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to add fields"
        )

# Extend existing endpoints to accept new fields
@router.post("/", response_model={ModelName}Response, status_code=status.HTTP_201_CREATED)
async def create_enhanced_{entity}(
    screen_data: Enhanced{ModelName}Create,  # Use enhanced model
    db: Session = Depends(get_db)
):
    """Create new {entity} with support for new fields"""
    try:
        # Convert to dictionary and handle new fields
        data_dict = screen_data.model_dump(exclude_none=True)
        
        # Validate enhanced screen data
        validate_enhanced_screen_data(data_dict)
        
        # Create with new field support
        created_item = await data_controller.create({ModelName}, data_dict)
        
        logger.info(f"Created enhanced {entity} with ID: {created_item.id}")
        return created_item
        
    except Exception as e:
        logger.error(f"Error creating enhanced {entity}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to create enhanced entity"
        )
```

**Database Migration Strategy - Schema Updates:**
```python
# Database migration for new fields (using Alembic or direct SQL)
async def add_new_fields_migration():
    """Add new columns to existing table"""
    
    migration_sql = """
    ALTER TABLE {table_name} 
    ADD COLUMN new_field1 VARCHAR(255),
    ADD COLUMN new_field2 VARCHAR(500),
    ADD COLUMN new_dropdown_field VARCHAR(50);
    """
    
    # Execute migration with proper error handling
    try:
        await execute_migration(migration_sql)
        logger.info("Successfully added new fields to database")
    except Exception as e:
        logger.error(f"Migration failed: {str(e)}")
        raise
```

#### Field Addition Validation - Quality Assurance

**Backward Compatibility Preservation:**
- All new fields must be optional to avoid breaking existing API contracts
- Existing form functionality must remain completely unchanged
- Database schema changes must be non-breaking (nullable columns)
- Existing validation logic must continue to work without modification

**Testing Strategy for Field Addition - Comprehensive Validation:**
```python
# Test new fields without affecting existing functionality
def test_add_fields_functionality():
    """Test that new fields work correctly"""
    
    # Test creation with new fields
    enhanced_data = {
        "existing_field": "value",  # existing fields preserved
        "new_field1": "new_value",
        "new_dropdown_field": "option1"
    }
    
    response = client.post("/api/service/", json=enhanced_data)
    assert response.status_code == 201
    assert response.json()["new_field1"] == "new_value"

def test_backward_compatibility():
    """Test that existing functionality still works"""
    
    # Test creation without new fields (should still work)
    existing_data = {
        "existing_field": "value"
    }
    
    response = client.post("/api/service/", json=existing_data)
    assert response.status_code == 201
    # New fields should be None/null but not cause errors
```

---

## TESTING AND VALIDATION - Quality Assurance Framework

## Comprehensive Testing Framework - Complete Test Coverage

### Test Suite Generation - Automated Testing
Create complete test coverage following `.github/chatmodes/Govern.chatmode.md` guidelines:

#### Frontend Component Testing - React Testing Library
```jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import '@testing-library/jest-dom';
import {ComponentName} from './{ComponentName}';

// Mock fetch for API testing
global.fetch = jest.fn();

const MockedComponent = () => (
    <BrowserRouter>
        <{ComponentName} />
    </BrowserRouter>
);

describe('{ComponentName}', () => {
    beforeEach(() => {
        fetch.mockClear();
    });
    
    test('renders component with all form fields', () => {
        render(<MockedComponent />);
        
        // Test all required form fields are present
        expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
        expect(screen.getByLabelText(/category/i)).toBeInTheDocument();
        expect(screen.getByRole('button', { name: /create/i })).toBeInTheDocument();
    });
    
    test('handles form submission successfully', async () => {
        fetch.mockResolvedValueOnce({
            ok: true,
            json: async () => ({ id: 1, name: 'Test Entity' })
        });
        
        render(<MockedComponent />);
        
        // Fill form fields
        fireEvent.change(screen.getByLabelText(/name/i), {
            target: { value: 'Test Entity' }
        });
        fireEvent.change(screen.getByLabelText(/category/i), {
            target: { value: 'option1' }
        });
        
        // Submit form
        fireEvent.click(screen.getByRole('button', { name: /create/i }));
        
        // Verify API call
        await waitFor(() => {
            expect(fetch).toHaveBeenCalledWith('/api/{servicename}/', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    name: 'Test Entity',
                    category: 'option1'
                })
            });
        });
        
        // Verify success message
        expect(await screen.findByText(/created successfully/i)).toBeInTheDocument();
    });
    
    test('handles API error responses', async () => {
        fetch.mockResolvedValueOnce({
            ok: false,
            json: async () => ({ detail: 'Validation error' })
        });
        
        render(<MockedComponent />);
        
        // Submit form
        fireEvent.click(screen.getByRole('button', { name: /create/i }));
        
        // Verify error message
        expect(await screen.findByText(/validation error/i)).toBeInTheDocument();
    });
    
    test('validates required fields', async () => {
        render(<MockedComponent />);
        
        // Try to submit without required fields
        fireEvent.click(screen.getByRole('button', { name: /create/i }));
        
        // Verify validation error
        expect(await screen.findByText(/name is required/i)).toBeInTheDocument();
    });
    
    test('handles loading states correctly', async () => {
        fetch.mockImplementationOnce(() => new Promise(resolve => 
            setTimeout(() => resolve({
                ok: true,
                json: async () => ({ id: 1 })
            }), 100)
        ));
        
        render(<MockedComponent />);
        
        // Fill and submit form
        fireEvent.change(screen.getByLabelText(/name/i), {
            target: { value: 'Test' }
        });
        fireEvent.click(screen.getByRole('button', { name: /create/i }));
        
        // Verify loading state
        expect(screen.getByText(/saving/i)).toBeInTheDocument();
        expect(screen.getByRole('button', { name: /saving/i })).toBeDisabled();
    });
});
```

#### Backend Service Testing - FastAPI TestClient
```python
import pytest
from fastapi.testclient import TestClient
from main import app
from tests.test_utils import TestDatabase

client = TestClient(app)

class Test{ScreenName}Service:
    """Comprehensive test suite for {ScreenName} service"""
    
    def setup_method(self):
        """Setup test database and data"""
        self.test_db = TestDatabase()
        self.valid_screen_data = {
            "name": "Test Entity",
            "category": "option1",
            "amount": 100.50,
            "description": "Test description"
        }
    
    def teardown_method(self):
        """Cleanup test data"""
        self.test_db.cleanup()
    
    def test_create_entity_with_screen_data(self):
        """Test entity creation with complete screen data"""
        response = client.post("/api/{servicename}/", json=self.valid_screen_data)
        assert response.status_code == 201
        
        data = response.json()
        assert data["id"] is not None
        assert data["name"] == "Test Entity"
        assert data["category"] == "option1"
    
    def test_data_type_validation(self):
        """Test data type validation for form fields"""
        invalid_data = self.valid_screen_data.copy()
        invalid_data["amount"] = "invalid_amount"  # Should be number
        
        response = client.post("/api/{servicename}/", json=invalid_data)
        assert response.status_code == 400
        assert "must be" in response.json()["detail"].lower()
    
    def test_field_constraint_validation(self):
        """Test field constraint validation"""
        invalid_data = self.valid_screen_data.copy()
        invalid_data["name"] = "x" * 101  # Exceeds max length
        
        response = client.post("/api/{servicename}/", json=invalid_data)
        assert response.status_code == 400
        assert "exceeds maximum length" in response.json()["detail"]
    
    def test_dropdown_value_validation(self):
        """Test dropdown/enum value validation"""
        invalid_data = self.valid_screen_data.copy()
        invalid_data["category"] = "invalid_option"
        
        response = client.post("/api/{servicename}/", json=invalid_data)
        assert response.status_code == 400
        assert "must be one of" in response.json()["detail"]
    
    def test_related_data_persistence(self):
        """Test that related entities are properly persisted"""
        screen_data = self.valid_screen_data.copy()
        screen_data["related_items"] = [
            {"name": "Related 1", "value": 10},
            {"name": "Related 2", "value": 20}
        ]
        
        response = client.post("/api/{servicename}/", json=screen_data)
        assert response.status_code == 201
        
        # Verify related data was saved
        entity_id = response.json()["id"]
        related_response = client.get(f"/api/{servicename}/{entity_id}")
        assert related_response.status_code == 200
        # Add assertions for related data presence
    
    def test_empty_request_handling(self):
        """Test handling of empty request body"""
        response = client.post("/api/{servicename}/", json={})
        assert response.status_code == 422
```

---

## Quality Assurance Framework - Standards Compliance

### Validation Checklist - Requirements Verification
Verify generated screens meet all requirements:

#### Frontend Structure Validation
- ✅ Screen directory exists in frontend/src/screens/{ComponentName}/
- ✅ Main component file {ComponentName}.js exists and exports default component
- ✅ Component follows React functional component pattern with proper hooks
- ✅ CSS file exists with same name as component and is imported
- ✅ Screen entry added to left-navigation.json with showInNavigation flag

#### React Component Validation
- ✅ Valid JSX syntax with proper closing tags and attributes
- ✅ Hooks called at top level of functional components only
- ✅ Optional chaining (?.) or proper null checks used for nested object properties
- ✅ All screen components have export default {ComponentName}
- ✅ State management properly implemented with useState and useEffect

#### API Integration Validation
- ✅ API calls implemented using established HTTP client patterns
- ✅ Response handling appropriate for chosen implementation
- ✅ Proper try-catch blocks around all async operations
- ✅ Loading state set to true before API calls, false after
- ✅ Error state properly set and cleared in catch blocks

#### Backend Service Validation
- ✅ Data persistence implemented for all entities collected on screen
- ✅ Related data entities properly processed and saved to database
- ✅ Data type validation implemented for all form fields
- ✅ Field constraints validated before database operations
- ✅ Dropdown/enum value validation implemented properly

#### Testing and Quality Validation
- ✅ Frontend component tested with mocked APIs and real backend integration
- ✅ Backend service tested with FastAPI TestClient including empty request scenarios
- ✅ Error handling tested at both frontend and backend layers
- ✅ Responsive design and accessibility compliance verified
- ✅ Navigation integration tested and verified

### Quality Standards - Framework Compliance
Ensure all generated screens meet framework standards:
- Complete implementations with no placeholder code
- Proper error handling with user-friendly messages and technical debug information
- Responsive design that works across different screen sizes
- Accessibility compliance with WCAG guidelines
- Performance optimization for data loading and form interactions
- Security considerations for data validation and API integration

<!-- COMPLETION VERIFICATION - Mode-Specific Success Criteria -->
Before marking screen generation complete, verify appropriate mode completion:

**For BUILD-SCREENS mode:**
☐ Complete React screen component with proper hooks and state management
☐ Backend service implements all required endpoints with data persistence
☐ All widget requirements implemented with comprehensive validation
☐ Navigation integration completed in left-navigation.json
☐ API integration follows established patterns with proper error handling
☐ Related entities properly processed and persisted to database
☐ Responsive design and accessibility standards met
☐ Comprehensive testing covers both frontend and backend functionality
☐ Service registration completed in main.py following patterns
☐ Form validation handles all field types and constraints properly

**For EXTEND-SCREENS mode:**
☐ New widget requirements successfully integrated into existing screen
☐ Enhanced functionality works seamlessly with existing component structure
☐ New backend endpoints follow existing patterns and maintain compatibility
☐ Existing screen functionality continues to work without regression
☐ API integration maintains backward compatibility for existing consumers
☐ Enhanced data persistence handles new entities and relationships
☐ Navigation updates integrate properly with existing routing
☐ Test coverage extended to include new functionality and edge cases
☐ Performance characteristics maintained or improved with enhancements
☐ Documentation updated to reflect new functionality

**For REBUILD-SCREENS mode:**
☐ All widget requirements implemented in fresh screen architecture
☐ Existing valuable custom logic successfully integrated into new structure
☐ Previous screen functionality preserved where not superseded by new requirements
☐ Fresh component follows updated widget specifications exactly
☐ Backend service architecture regenerated with enhanced data persistence
☐ Custom styling and interaction patterns preserved where beneficial
☐ Navigation integration maintains compatibility with existing routing
☐ Performance characteristics maintained or improved compared to previous implementation
☐ Migration strategy planned for transitioning to regenerated screen
☐ Comprehensive testing validates all regenerated functionality

**For ADD-FIELDS mode:**
☐ New fields successfully integrated into existing screen without breaking functionality
☐ Backend models extended with optional new fields maintaining backward compatibility
☐ Database migration completed for new columns with proper nullable constraints
☐ Enhanced API endpoints accept new fields while preserving existing contracts
☐ Existing form functionality continues to work exactly as before
☐ New field validation works correctly without affecting existing validation
☐ Frontend state management handles new fields seamlessly with existing state
☐ API integration maintains full backward compatibility for existing consumers
☐ Enhanced form submission combines old and new data correctly
☐ Testing covers both new field functionality and existing feature preservation

**Universal verification for all modes:**
☐ Screen component follows React best practices and established patterns
☐ Backend API endpoints properly documented with request/response examples
☐ Database operations optimized with proper DataAccessController usage
☐ Error handling provides sufficient detail for debugging and user guidance
☐ Widget requirements accurately implemented throughout full stack
☐ Security measures appropriate for screen operations and data handling
☐ Code follows established project patterns and naming conventions
☐ Integration with existing frontend and backend infrastructure functions correctly
<!-- END VERIFICATION -->

---

## ERROR HANDLING AND TROUBLESHOOTING - Issue Resolution

### Common Issues and Solutions - Problem Diagnostics

**React Component Issues:**
- **Hooks Rules Violations**: Ensure hooks are called at top level only
- **State Management Issues**: Use proper useState and useEffect patterns
- **API Integration Errors**: Implement proper try-catch blocks around async operations
- **Routing Issues**: Verify navigation integration with left-navigation.json

**FastAPI Service Issues:**
- **DataAccessController Issues**: Ensure .model_dump() conversion before database operations
- **API Endpoint Errors**: Verify proper router registration in main.py
- **Validation Errors**: Align Pydantic models with frontend form requirements
- **Database Integration Issues**: Check SQLAlchemy model relationships and constraints

**Full-Stack Integration Issues:**
- **API Contract Mismatches**: Ensure frontend and backend data structures align
- **Navigation Integration**: Verify screen registration in left-navigation.json
- **Service Registration**: Confirm service import and router registration in main.py
- **Database Persistence**: Validate related entity processing and data relationships

### Quality Standards - Framework Standards
Ensure all generated code meets framework standards:
- Complete implementations with no placeholder comments
- Proper error handling with user-friendly messages and technical debug information
- Responsive design that works across different screen sizes
- Accessibility compliance with WCAG guidelines
- Performance optimization for data loading and form interactions
- Security considerations for data validation and API integration

## Success Criteria - Implementation Goals
- Screens successfully integrate with existing navigation and routing system
- All widget requirements accurately implemented in React components
- Complete FastAPI service set supports full CRUD operations with proper validation
- Backend services properly registered and integrated with DataAccessController
- Database relationships correctly established for all collected entities
- Comprehensive testing validates both frontend and backend functionality
- Code follows established project patterns and naming conventions

Execute appropriate screen generation mode based on user command context, ensuring comprehensive full-stack implementation of widget requirements while maintaining seamless integration with React + FastAPI + SQLAlchemy + DataAccessController technology stack.
