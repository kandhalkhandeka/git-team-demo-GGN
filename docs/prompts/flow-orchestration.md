# FLOW ORCHESTRATION - Multi-Step Flow Development

## Command Patterns
When user inputs any of:
- `build flow`
- `create multi-step flow`
- `generate flow with wizard`
- `build react-use-wizard flow`
- `extend flow`
- `merge steps`

## Purpose
Generate comprehensive multi-step flows using react-use-wizard with backend step services and data persistence, following established project patterns with complete flow orchestration and testing.

## Prerequisites
- Domain models have been created for flow and step data
- Widget flow requirements and step breakdown are defined
- react-use-wizard dependency is available in package.json
- Existing flow patterns have been analyzed

## Mode Detection
Automatically detect execution mode based on context:
- **build-flow**: Fresh flow generation with react-use-wizard integration
- **extend-flow**: Add new steps and functionality to existing flows
- **merge-steps**: Consolidate multiple flow steps while preserving custom logic

---

## DISCOVERY AND ANALYSIS - Flow Architecture Analysis

### Flow Architecture Discovery - Pattern Recognition
Analyze existing codebase to understand established patterns:

1. **Examine Flow Documentation**
   - Check `README.md` and `docs/README.md` (if available) for widget specifications, flow patterns, and react-use-wizard setup
   - Search workspace for flow architecture documentation files
   - Extract flow creation patterns and react-use-wizard usage conventions
   - Identify integration requirements and established guidelines
   - Document multi-step flow development standards

2. **Analyze Existing Flows**
   - Review all flow implementations and step components
   - Extract flow structure patterns and navigation conventions
   - Identify react-use-wizard integration patterns and step orchestration
   - Document data persistence strategies and normalized table architecture
   - Collect best practices from existing flow implementations

3. **Study Integration Requirements**
   - Locate and analyze navigation registration patterns
   - Understand flow service integration and API requirements
   - Document step-specific validation and business rule patterns
   - Extract testing patterns and quality assurance requirements

### Consolidation and Planning - Flow Implementation Strategy
Consolidate all discovered patterns into a comprehensive flow generation strategy that maintains consistency with existing codebase while implementing new multi-step flow requirements.

---

## FLOW GENERATION EXECUTION - React-Use-Wizard Implementation

## BUILD-FLOW MODE

### Purpose
Generate complete multi-step flow with react-use-wizard integration, backend step services, and normalized data persistence.

### Multi-Step Flow Creation - Component Architecture
Generate complete flow implementations following discovered patterns:

#### Backend Flow Architecture - Service Layer Generation
Generate comprehensive backend supporting flow functionality:

##### Flow Control Model - State Management Schema
```python
from sqlalchemy import Column, Integer, String, DateTime, Boolean, Text, ForeignKey, JSON
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
from pydantic import BaseModel, Field
from typing import Optional, Dict, Any
from datetime import datetime
from models.base import Base

class FlowControl(Base):
    __tablename__ = "flow_control"
    __table_args__ = {'extend_existing': True}
    
    # Primary key
    id = Column(Integer, primary_key=True, index=True)
    
    # Flow identification and state
    flow_type = Column(String(100), nullable=False)
    current_step = Column(Integer, default=1, nullable=False)
    total_steps = Column(Integer, nullable=False)
    flow_status = Column(String(50), default='in_progress', nullable=False)  # in_progress, completed, cancelled
    
    # Flow metadata
    user_id = Column(String(100), nullable=True)  # Optional user tracking
    flow_data = Column(JSON, nullable=True)  # Additional flow metadata
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now(), nullable=False)
    updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False)
    completed_at = Column(DateTime(timezone=True), nullable=True)
    
    # Relationships to step data
    step_data = relationship("FlowStepData", back_populates="flow", cascade="all, delete-orphan")

# Base class for step-specific data
class FlowStepData(Base):
    __tablename__ = "flow_step_data"
    __table_args__ = {'extend_existing': True}
    
    id = Column(Integer, primary_key=True, index=True)
    flow_id = Column(Integer, ForeignKey('flow_control.id'), nullable=False)
    step_number = Column(Integer, nullable=False)
    step_name = Column(String(100), nullable=False)
    step_data = Column(JSON, nullable=False)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now(), nullable=False)
    updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False)
    
    # Relationship
    flow = relationship("FlowControl", back_populates="step_data")

# Step-specific normalized models (generate based on requirements)
class Step1Data(Base):
    __tablename__ = "step1_data"
    __table_args__ = {'extend_existing': True}
    
    id = Column(Integer, primary_key=True, index=True)
    flow_id = Column(Integer, ForeignKey('flow_control.id'), nullable=False)
    
    # Step-specific fields based on widget requirements
    field1 = Column(String(255), nullable=False)
    field2 = Column(Integer, nullable=True)
    field3 = Column(Boolean, default=False, nullable=False)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now(), nullable=False)
    updated_at = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False)

# Pydantic schemas for flow control
class FlowControlBase(BaseModel):
    flow_type: str = Field(..., max_length=100, description="Type of flow")
    total_steps: int = Field(..., ge=1, description="Total number of steps in flow")
    user_id: Optional[str] = Field(None, max_length=100, description="Optional user identifier")

class FlowControlCreate(FlowControlBase):
    pass

class FlowControlUpdate(BaseModel):
    current_step: Optional[int] = Field(None, ge=1, description="Current step number")
    flow_status: Optional[str] = Field(None, description="Flow status")
    flow_data: Optional[Dict[str, Any]] = Field(None, description="Additional flow metadata")

class FlowControlResponse(FlowControlBase):
    id: int
    current_step: int
    flow_status: str
    flow_data: Optional[Dict[str, Any]]
    created_at: datetime
    updated_at: datetime
    completed_at: Optional[datetime]
    
    class Config:
        from_attributes = True

# Step-specific schemas
class Step1DataBase(BaseModel):
    field1: str = Field(..., max_length=255, description="Step 1 field 1")
    field2: Optional[int] = Field(None, description="Step 1 field 2")
    field3: bool = Field(False, description="Step 1 field 3")

class Step1DataCreate(Step1DataBase):
    flow_id: int = Field(..., description="Associated flow ID")

class Step1DataResponse(Step1DataBase):
    id: int
    flow_id: int
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True
```

##### Flow Orchestration Service - API Coordination
```python
from fastapi import APIRouter, HTTPException, Depends, status
from sqlalchemy.orm import Session
from controllers.DataAccessController import data_controller
from models.FlowControl import FlowControl, FlowControlCreate, FlowControlUpdate, FlowControlResponse
from models.FlowStepData import FlowStepData
from database import get_db
from typing import List, Dict, Any
import logging

# Configure logging
logger = logging.getLogger(__name__)

# Create router for flow orchestration
router = APIRouter(
    prefix="/api/flow",
    tags=["flow-orchestration"]
)

@router.post("/initialize", response_model=FlowControlResponse, status_code=status.HTTP_201_CREATED)
async def initialize_flow(
    flow_data: FlowControlCreate,
    db: Session = Depends(get_db)
):
    """Initialize new flow with proper state management"""
    try:
        # Convert Pydantic model to dictionary
        flow_dict = flow_data.model_dump()
        flow_dict['current_step'] = 1
        flow_dict['flow_status'] = 'in_progress'
        
        # Create flow control record
        flow_control = await data_controller.create(FlowControl, flow_dict)
        logger.info(f"Initialized flow with ID: {flow_control.id}")
        
        return flow_control
        
    except Exception as e:
        logger.error(f"Error initializing flow: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to initialize flow"
        )

@router.get("/{flow_id}", response_model=FlowControlResponse)
async def get_flow_state(flow_id: int, db: Session = Depends(get_db)):
    """Get current flow state and progress"""
    try:
        flow = await data_controller.get_by_id(FlowControl, flow_id)
        if not flow:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"Flow with ID {flow_id} not found"
            )
        
        return flow
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error retrieving flow {flow_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to retrieve flow state"
        )

@router.put("/{flow_id}/step", response_model=FlowControlResponse)
async def update_flow_step(
    flow_id: int,
    step_update: FlowControlUpdate,
    db: Session = Depends(get_db)
):
    """Update flow step and state"""
    try:
        # Verify flow exists
        existing_flow = await data_controller.get_by_id(FlowControl, flow_id)
        if not existing_flow:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"Flow with ID {flow_id} not found"
            )
        
        # Update flow state
        update_dict = {k: v for k, v in step_update.model_dump().items() if v is not None}
        
        # Handle flow completion
        if update_dict.get('flow_status') == 'completed':
            update_dict['completed_at'] = datetime.utcnow()
        
        updated_flow = await data_controller.update(FlowControl, flow_id, update_dict)
        logger.info(f"Updated flow {flow_id} to step {updated_flow.current_step}")
        
        return updated_flow
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error updating flow step {flow_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to update flow step"
        )

@router.post("/{flow_id}/consolidate", status_code=status.HTTP_200_OK)
async def consolidate_flow_data(flow_id: int, db: Session = Depends(get_db)):
    """Consolidate all step data and complete flow"""
    try:
        # Verify flow exists
        flow = await data_controller.get_by_id(FlowControl, flow_id)
        if not flow:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"Flow with ID {flow_id} not found"
            )
        
        # Collect all step data
        consolidated_data = await collect_all_step_data(flow_id)
        
        # Process consolidated data (implement business logic)
        result = await process_consolidated_data(consolidated_data)
        
        # Mark flow as completed
        await data_controller.update(FlowControl, flow_id, {
            'flow_status': 'completed',
            'completed_at': datetime.utcnow(),
            'flow_data': {'consolidation_result': result}
        })
        
        logger.info(f"Consolidated and completed flow {flow_id}")
        return {"status": "success", "message": "Flow completed successfully", "result": result}
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error consolidating flow {flow_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to consolidate flow data"
        )

async def collect_all_step_data(flow_id: int) -> Dict[str, Any]:
    """Collect data from all step tables"""
    consolidated_data = {}
    
    try:
        # Collect from each step table (generate based on flow requirements)
        step1_data = await data_controller.get_by_field(Step1Data, 'flow_id', flow_id)
        if step1_data:
            consolidated_data['step1'] = step1_data[0].__dict__
        
        # Add more steps as needed
        # step2_data = await data_controller.get_by_field(Step2Data, 'flow_id', flow_id)
        # if step2_data:
        #     consolidated_data['step2'] = step2_data[0].__dict__
        
        return consolidated_data
        
    except Exception as e:
        logger.error(f"Error collecting step data for flow {flow_id}: {str(e)}")
        raise

async def process_consolidated_data(data: Dict[str, Any]) -> Dict[str, Any]:
    """Process consolidated data according to business requirements"""
    try:
        # Implement business logic for data processing
        processed_result = {
            'total_steps_completed': len(data),
            'processing_timestamp': datetime.utcnow().isoformat(),
            'processed_data': data
        }
        
        return processed_result
        
    except Exception as e:
        logger.error(f"Error processing consolidated data: {str(e)}")
        raise
```

##### Step-Specific Services - Dedicated Step APIs
```python
# Step 1 Service
from fastapi import APIRouter, HTTPException, Depends, status
from sqlalchemy.orm import Session
from controllers.DataAccessController import data_controller
from models.Step1Data import Step1Data, Step1DataCreate, Step1DataResponse
from database import get_db
import logging

logger = logging.getLogger(__name__)

router = APIRouter(
    prefix="/api/step1",
    tags=["step1"]
)

@router.post("/", response_model=Step1DataResponse, status_code=status.HTTP_201_CREATED)
async def save_step1_data(
    step_data: Step1DataCreate,
    db: Session = Depends(get_db)
):
    """Save step 1 data with validation"""
    try:
        # Convert to dictionary for DataAccessController
        data_dict = step_data.model_dump()
        
        # Validate step data according to business rules
        validate_step1_data(data_dict)
        
        # Check if data already exists for this flow
        existing_data = await data_controller.get_by_field(Step1Data, 'flow_id', data_dict['flow_id'])
        
        if existing_data:
            # Update existing data
            updated_data = await data_controller.update(Step1Data, existing_data[0].id, data_dict)
            logger.info(f"Updated step 1 data for flow {data_dict['flow_id']}")
            return updated_data
        else:
            # Create new data
            created_data = await data_controller.create(Step1Data, data_dict)
            logger.info(f"Created step 1 data for flow {data_dict['flow_id']}")
            return created_data
        
    except ValueError as e:
        logger.error(f"Validation error in step 1: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Validation error: {str(e)}"
        )
    except Exception as e:
        logger.error(f"Error saving step 1 data: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to save step 1 data"
        )

@router.get("/{flow_id}", response_model=Step1DataResponse)
async def get_step1_data(flow_id: int, db: Session = Depends(get_db)):
    """Retrieve step 1 data for flow"""
    try:
        step_data = await data_controller.get_by_field(Step1Data, 'flow_id', flow_id)
        
        if not step_data:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail=f"Step 1 data not found for flow {flow_id}"
            )
        
        return step_data[0]
        
    except HTTPException:
        raise
    except Exception as e:
        logger.error(f"Error retrieving step 1 data for flow {flow_id}: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Failed to retrieve step 1 data"
        )

def validate_step1_data(data: Dict[str, Any]) -> None:
    """Validate step 1 data according to business rules"""
    
    # Required field validation
    if not data.get('field1', '').strip():
        raise ValueError("Field 1 is required")
    
    # Data type validation
    if data.get('field2') is not None and not isinstance(data['field2'], int):
        raise ValueError("Field 2 must be an integer")
    
    # Business rule validation
    if data.get('field2', 0) < 0:
        raise ValueError("Field 2 cannot be negative")
    
    # Additional validation based on widget requirements
    if len(data.get('field1', '')) > 255:
        raise ValueError("Field 1 cannot exceed 255 characters")
```

#### Frontend Flow Implementation with react-use-wizard - Component Development

##### Flow Container Component - Wizard Orchestration
```jsx
import React, { useState, useEffect } from 'react';
import { Wizard } from 'react-use-wizard';
import { Box, Paper, Typography, Alert, CircularProgress } from '@mui/material';
import { useNavigate, useParams } from 'react-router-dom';
import Step1Component from './steps/Step1Component';
import Step2Component from './steps/Step2Component';
import Step3Component from './steps/Step3Component';
import './FlowContainer.css';

const FlowContainer = () => {
    // Flow state management
    const [flowId, setFlowId] = useState(null);
    const [flowData, setFlowData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState('');
    
    // Navigation
    const navigate = useNavigate();
    const { id } = useParams();
    const isResumeMode = Boolean(id);
    
    // Initialize or resume flow
    useEffect(() => {
        if (isResumeMode) {
            resumeFlow(id);
        } else {
            initializeFlow();
        }
    }, [id]);
    
    const initializeFlow = async () => {
        setLoading(true);
        setError('');
        
        try {
            const response = await fetch('/api/flow/initialize', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    flow_type: 'widget_flow',
                    total_steps: 3, // Adjust based on flow requirements
                    user_id: 'current_user' // Implement user identification
                })
            });
            
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.detail || 'Failed to initialize flow');
            }
            
            const flowData = await response.json();
            setFlowId(flowData.id);
            setFlowData(flowData);
            
        } catch (err) {
            console.error('Error initializing flow:', err);
            setError(`Failed to initialize flow: ${err.message}`);
        } finally {
            setLoading(false);
        }
    };
    
    const resumeFlow = async (flowId) => {
        setLoading(true);
        setError('');
        
        try {
            const response = await fetch(`/api/flow/${flowId}`);
            
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.detail || 'Failed to resume flow');
            }
            
            const flowData = await response.json();
            setFlowId(flowData.id);
            setFlowData(flowData);
            
        } catch (err) {
            console.error('Error resuming flow:', err);
            setError(`Failed to resume flow: ${err.message}`);
        } finally {
            setLoading(false);
        }
    };
    
    const updateFlowStep = async (stepNumber) => {
        try {
            await fetch(`/api/flow/${flowId}/step`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    current_step: stepNumber
                })
            });
        } catch (err) {
            console.error('Error updating flow step:', err);
        }
    };
    
    const completeFlow = async () => {
        setLoading(true);
        
        try {
            const response = await fetch(`/api/flow/${flowId}/consolidate`, {
                method: 'POST'
            });
            
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.detail || 'Failed to complete flow');
            }
            
            const result = await response.json();
            
            // Navigate to success page or flow listing
            navigate('/flow-success', { state: { result } });
            
        } catch (err) {
            console.error('Error completing flow:', err);
            setError(`Failed to complete flow: ${err.message}`);
        } finally {
            setLoading(false);
        }
    };
    
    const handleStepChange = ({ activeStep, previousStep }) => {
        // Update backend with current step
        updateFlowStep(activeStep + 1);
        
        // Update local state
        setFlowData(prev => ({
            ...prev,
            current_step: activeStep + 1
        }));
    };
    
    if (loading && !flowId) {
        return (
            <Box display="flex" justifyContent="center" alignItems="center" minHeight="400px">
                <CircularProgress />
                <Typography variant="h6" sx={{ ml: 2 }}>
                    {isResumeMode ? 'Resuming flow...' : 'Initializing flow...'}
                </Typography>
            </Box>
        );
    }
    
    if (error) {
        return (
            <Box className="flow-container">
                <Paper elevation={3} sx={{ p: 3 }}>
                    <Alert severity="error">
                        <Typography variant="h6">Flow Error</Typography>
                        <Typography>{error}</Typography>
                    </Alert>
                </Paper>
            </Box>
        );
    }
    
    return (
        <Box className="flow-container">
            <Paper elevation={3} className="flow-paper">
                <Typography variant="h4" component="h1" gutterBottom>
                    Widget Flow Process
                </Typography>
                
                <Typography variant="body1" color="text.secondary" gutterBottom>
                    Step {flowData?.current_step || 1} of {flowData?.total_steps || 3}
                </Typography>
                
                <Wizard
                    startIndex={flowData?.current_step ? flowData.current_step - 1 : 0}
                    onStepChange={handleStepChange}
                >
                    <Step1Component 
                        flowId={flowId}
                        onStepComplete={() => console.log('Step 1 completed')}
                    />
                    <Step2Component 
                        flowId={flowId}
                        onStepComplete={() => console.log('Step 2 completed')}
                    />
                    <Step3Component 
                        flowId={flowId}
                        onStepComplete={() => console.log('Step 3 completed')}
                        onFlowComplete={completeFlow}
                    />
                </Wizard>
            </Paper>
        </Box>
    );
};

export default FlowContainer;
```

##### Step Component with react-use-wizard Integration - Step Management
```jsx
import React, { useState, useEffect } from 'react';
import { useWizard } from 'react-use-wizard';
import {
    Box,
    Typography,
    TextField,
    Button,
    FormControl,
    InputLabel,
    Select,
    MenuItem,
    Alert,
    CircularProgress,
    Grid
} from '@mui/material';

const Step1Component = ({ flowId, onStepComplete }) => {
    // react-use-wizard hooks
    const { nextStep, previousStep, isFirstStep, isLastStep, activeStep } = useWizard();
    
    // Component state
    const [formData, setFormData] = useState({
        field1: '',
        field2: '',
        field3: false
    });
    
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState('');
    const [dataLoaded, setDataLoaded] = useState(false);
    
    // Load existing data when component mounts
    useEffect(() => {
        if (flowId && !dataLoaded) {
            loadStepData();
        }
    }, [flowId]);
    
    const loadStepData = async () => {
        try {
            const response = await fetch(`/api/step1/${flowId}`);
            
            if (response.ok) {
                const existingData = await response.json();
                setFormData({
                    field1: existingData.field1 || '',
                    field2: existingData.field2 || '',
                    field3: existingData.field3 || false
                });
            }
            // If no data exists (404), continue with empty form
            
        } catch (err) {
            console.error('Error loading step data:', err);
            // Continue with empty form
        } finally {
            setDataLoaded(true);
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
    
    const validateForm = () => {
        if (!formData.field1.trim()) {
            throw new Error('Field 1 is required');
        }
        
        if (formData.field2 && isNaN(formData.field2)) {
            throw new Error('Field 2 must be a number');
        }
        
        if (formData.field2 && formData.field2 < 0) {
            throw new Error('Field 2 cannot be negative');
        }
    };
    
    const saveStepData = async () => {
        setLoading(true);
        setError('');
        
        try {
            validateForm();
            
            const stepData = {
                ...formData,
                flow_id: flowId,
                field2: formData.field2 ? parseInt(formData.field2) : null
            };
            
            const response = await fetch('/api/step1/', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(stepData)
            });
            
            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.detail || 'Failed to save step data');
            }
            
            // Call completion callback
            if (onStepComplete) {
                onStepComplete();
            }
            
            return true;
            
        } catch (err) {
            console.error('Error saving step data:', err);
            setError(err.message);
            return false;
        } finally {
            setLoading(false);
        }
    };
    
    const handleNext = async () => {
        const saved = await saveStepData();
        if (saved) {
            nextStep();
        }
    };
    
    const handlePrevious = () => {
        previousStep();
    };
    
    const handleSaveAndContinueLater = async () => {
        const saved = await saveStepData();
        if (saved) {
            // Navigate to flow listing or dashboard
            window.location.href = '/flows';
        }
    };
    
    return (
        <Box className="step-container">
            <Typography variant="h5" component="h2" gutterBottom>
                Step 1: Basic Information
            </Typography>
            
            <Typography variant="body1" color="text.secondary" gutterBottom>
                Please provide the required information for step 1.
            </Typography>
            
            {error && (
                <Alert severity="error" sx={{ mb: 3 }}>
                    {error}
                </Alert>
            )}
            
            <Grid container spacing={3}>
                <Grid item xs={12} md={6}>
                    <TextField
                        fullWidth
                        label="Field 1"
                        value={formData.field1}
                        onChange={handleInputChange('field1')}
                        required
                        error={!formData.field1.trim() && error}
                        helperText={!formData.field1.trim() && error ? 'Field 1 is required' : ''}
                    />
                </Grid>
                
                <Grid item xs={12} md={6}>
                    <TextField
                        fullWidth
                        label="Field 2"
                        type="number"
                        value={formData.field2}
                        onChange={handleInputChange('field2')}
                        inputProps={{ min: 0 }}
                        helperText="Optional numeric field"
                    />
                </Grid>
                
                <Grid item xs={12}>
                    <FormControl fullWidth>
                        <InputLabel>Field 3</InputLabel>
                        <Select
                            value={formData.field3}
                            onChange={handleInputChange('field3')}
                            label="Field 3"
                        >
                            <MenuItem value={true}>Yes</MenuItem>
                            <MenuItem value={false}>No</MenuItem>
                        </Select>
                    </FormControl>
                </Grid>
            </Grid>
            
            <Box sx={{ mt: 4, display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                <Box>
                    {!isFirstStep && (
                        <Button
                            variant="outlined"
                            onClick={handlePrevious}
                            disabled={loading}
                        >
                            Previous
                        </Button>
                    )}
                </Box>
                
                <Box sx={{ display: 'flex', gap: 2 }}>
                    <Button
                        variant="outlined"
                        onClick={handleSaveAndContinueLater}
                        disabled={loading}
                    >
                        Save & Continue Later
                    </Button>
                    
                    <Button
                        variant="contained"
                        onClick={handleNext}
                        disabled={loading}
                        startIcon={loading ? <CircularProgress size={20} /> : null}
                    >
                        {loading ? 'Saving...' : 'Next'}
                    </Button>
                </Box>
            </Box>
        </Box>
    );
};

export default Step1Component;
```

### Navigation Registration - Flow Integration
Register flow in left-navigation.json:

```json
{
    "path": "/flow",
    "name": "Widget Flow",
    "component": "FlowContainer",
    "showInNavigation": true,
    "icon": "FlowIcon",
    "order": 15
}
```

### Service Registration - Backend Registration
Register all services in main.py:

```python
# Flow services registration in main.py
from services.FlowOrchestrationService import router as flow_router
from services.Step1Service import router as step1_router
from services.Step2Service import router as step2_router
from services.Step3Service import router as step3_router

app.include_router(flow_router)
app.include_router(step1_router)
app.include_router(step2_router)
app.include_router(step3_router)
```

---

## EXTEND-FLOW MODE

### Purpose
Add new steps and functionality to existing react-use-wizard flows while maintaining backward compatibility and established patterns.

### Existing Flow Analysis - Architecture Assessment
Before implementing extensions, analyze current flow structure:
1. **Flow Architecture Review**: Analyze current Wizard container and step components
2. **Step Service Analysis**: Review existing step services and data models
3. **Navigation Flow Review**: Document current step progression and validation
4. **Data Persistence Analysis**: Understand current normalized data architecture
5. **Testing Coverage Assessment**: Evaluate current test coverage and patterns

#### Safe Extension Implementation - Non-Breaking Enhancement

**Add New Steps to Existing Flow:**
```jsx
// Enhanced Flow Container with additional steps
const EnhancedFlowContainer = () => {
    // Preserve existing state and functionality
    const [flowId, setFlowId] = useState(null);
    const [flowData, setFlowData] = useState(null);
    
    // Add new step tracking
    const totalSteps = 5; // Increased from original 3
    
    // Enhanced initialization for additional steps
    const initializeFlow = async () => {
        // Preserve existing initialization logic
        const response = await fetch('/api/flow/initialize', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                flow_type: 'enhanced_widget_flow',
                total_steps: totalSteps, // Updated total
                user_id: 'current_user'
            })
        });
        
        // Continue with existing logic
    };
    
    return (
        <Box className="flow-container">
            <Wizard
                startIndex={flowData?.current_step ? flowData.current_step - 1 : 0}
                onStepChange={handleStepChange}
            >
                {/* Preserve existing steps */}
                <Step1Component flowId={flowId} />
                <Step2Component flowId={flowId} />
                <Step3Component flowId={flowId} />
                
                {/* Add new steps */}
                <Step4Component flowId={flowId} />
                <Step5Component flowId={flowId} />
            </Wizard>
        </Box>
    );
};
```

**Extend Backend with New Step Services - Service Expansion:**
```python
# New Step 4 Service
@router.post("/", response_model=Step4DataResponse, status_code=status.HTTP_201_CREATED)
async def save_step4_data(step_data: Step4DataCreate, db: Session = Depends(get_db)):
    """Save step 4 data following established patterns"""
    try:
        # Follow existing validation and persistence patterns
        data_dict = step_data.model_dump()
        validate_step4_data(data_dict)
        
        # Use same create/update logic as existing steps
        existing_data = await data_controller.get_by_field(Step4Data, 'flow_id', data_dict['flow_id'])
        
        if existing_data:
            return await data_controller.update(Step4Data, existing_data[0].id, data_dict)
        else:
            return await data_controller.create(Step4Data, data_dict)
            
    except Exception as e:
        logger.error(f"Error saving step 4 data: {str(e)}")
        raise HTTPException(status_code=500, detail="Failed to save step 4 data")
```

**Enhanced Data Consolidation - Multi-Step Data Processing:**
```python
# Enhanced consolidation to include new steps
async def collect_all_step_data(flow_id: int) -> Dict[str, Any]:
    """Collect data from all step tables including new steps"""
    consolidated_data = {}
    
    # Preserve existing step data collection
    step1_data = await data_controller.get_by_field(Step1Data, 'flow_id', flow_id)
    if step1_data:
        consolidated_data['step1'] = step1_data[0].__dict__
    
    # Add new step data collection
    step4_data = await data_controller.get_by_field(Step4Data, 'flow_id', flow_id)
    if step4_data:
        consolidated_data['step4'] = step4_data[0].__dict__
    
    step5_data = await data_controller.get_by_field(Step5Data, 'flow_id', flow_id)
    if step5_data:
        consolidated_data['step5'] = step5_data[0].__dict__
    
    return consolidated_data
```

---

## MERGE-STEPS MODE

### Purpose
Consolidate multiple flow steps while preserving valuable custom logic and maintaining react-use-wizard integration patterns.

### Step Consolidation Analysis - Custom Logic Preservation
Before regenerating, systematically identify and preserve:
1. **Custom Step Logic**: User-defined validation and business rules in step components
2. **Flow Orchestration Customizations**: Custom navigation patterns and step progression logic
3. **Data Processing Logic**: Custom data transformation and consolidation functions
4. **Integration Patterns**: Custom API integration and error handling implementations
5. **Performance Optimizations**: Custom caching and state management optimizations

#### Regeneration with Preservation - Enhanced Flow Architecture

**Fresh Flow Architecture:**
- Generate new flow structure based on updated widget requirements
- Implement enhanced step progression and validation from updated specifications
- Create optimized data persistence architecture for current requirements
- Integrate preserved custom logic where compatible with new structure
- Maintain existing react-use-wizard patterns and navigation conventions

**Custom Logic Integration - Flow Logic Preservation:**
```jsx
// Preserve custom logic in regenerated flow
const RegeneratedFlowContainer = () => {
    // Preserve existing custom configurations
    const [customSettings, setCustomSettings] = useState(preserveCustomSettings());
    
    // Apply new flow structure with preserved orchestration logic
    const handleEnhancedStepChange = ({ activeStep, previousStep }) => {
        // Apply new step progression rules
        const newStepValidation = validateNewStepProgression(activeStep);
        
        // Apply preserved custom logic
        const customStepLogic = applyPreservedStepLogic(activeStep);
        
        // Standard flow update
        updateFlowStep(activeStep + 1);
    };
    
    // Enhanced completion with preserved logic
    const completeEnhancedFlow = async () => {
        // Apply new consolidation requirements
        const newConsolidation = await consolidateNewFlowData();
        
        // Apply preserved custom processing
        const customProcessing = await applyPreservedProcessing(newConsolidation);
        
        // Complete flow with enhanced logic
        return await finalizeEnhancedFlow(customProcessing);
    };
    
    return (
        <Wizard onStepChange={handleEnhancedStepChange}>
            {/* New step structure based on updated requirements */}
            {renderEnhancedSteps()}
        </Wizard>
    );
};
```

---

## TESTING AND VALIDATION - Flow Quality Assurance

## Comprehensive Testing Framework - React-Use-Wizard Testing

### Test Suite Generation - Automated Flow Testing
Create complete test coverage following `.github/chatmodes/Govern.chatmode.md` guidelines:

#### Flow Orchestration Testing - Wizard Integration Testing
```jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import '@testing-library/jest-dom';
import FlowContainer from './FlowContainer';

// Mock fetch for API testing
global.fetch = jest.fn();

const MockedFlowContainer = () => (
    <BrowserRouter>
        <FlowContainer />
    </BrowserRouter>
);

describe('FlowContainer', () => {
    beforeEach(() => {
        fetch.mockClear();
    });
    
    test('initializes flow on mount', async () => {
        fetch.mockResolvedValueOnce({
            ok: true,
            json: async () => ({
                id: 1,
                flow_type: 'widget_flow',
                current_step: 1,
                total_steps: 3,
                flow_status: 'in_progress'
            })
        });
        
        render(<MockedFlowContainer />);
        
        // Verify flow initialization API call
        await waitFor(() => {
            expect(fetch).toHaveBeenCalledWith('/api/flow/initialize', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    flow_type: 'widget_flow',
                    total_steps: 3,
                    user_id: 'current_user'
                })
            });
        });
        
        // Verify flow container renders
        expect(screen.getByText('Widget Flow Process')).toBeInTheDocument();
        expect(screen.getByText('Step 1 of 3')).toBeInTheDocument();
    });
    
    test('handles flow initialization error', async () => {
        fetch.mockResolvedValueOnce({
            ok: false,
            json: async () => ({ detail: 'Initialization failed' })
        });
        
        render(<MockedFlowContainer />);
        
        // Verify error message appears
        expect(await screen.findByText(/failed to initialize flow/i)).toBeInTheDocument();
    });
    
    test('resumes existing flow when ID provided', async () => {
        fetch.mockResolvedValueOnce({
            ok: true,
            json: async () => ({
                id: 123,
                current_step: 2,
                total_steps: 3,
                flow_status: 'in_progress'
            })
        });
        
        // Mock useParams to return flow ID
        jest.doMock('react-router-dom', () => ({
            ...jest.requireActual('react-router-dom'),
            useParams: () => ({ id: '123' })
        }));
        
        render(<MockedFlowContainer />);
        
        // Verify resume API call
        await waitFor(() => {
            expect(fetch).toHaveBeenCalledWith('/api/flow/123');
        });
    });
});
```

#### Step Component Testing - React-Use-Wizard Hook Testing
```jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { Wizard } from 'react-use-wizard';
import '@testing-library/jest-dom';
import Step1Component from './Step1Component';

// Mock useWizard hook
const mockUseWizard = {
    nextStep: jest.fn(),
    previousStep: jest.fn(),
    isFirstStep: true,
    isLastStep: false,
    activeStep: 0
};

jest.mock('react-use-wizard', () => ({
    useWizard: () => mockUseWizard
}));

global.fetch = jest.fn();

const MockedStep1Component = () => (
    <Wizard>
        <Step1Component flowId={1} onStepComplete={jest.fn()} />
    </Wizard>
);

describe('Step1Component', () => {
    beforeEach(() => {
        fetch.mockClear();
        mockUseWizard.nextStep.mockClear();
    });
    
    test('loads existing step data on mount', async () => {
        fetch.mockResolvedValueOnce({
            ok: true,
            json: async () => ({
                id: 1,
                flow_id: 1,
                field1: 'existing value',
                field2: 42,
                field3: true
            })
        });
        
        render(<MockedStep1Component />);
        
        // Verify API call to load data
        await waitFor(() => {
            expect(fetch).toHaveBeenCalledWith('/api/step1/1');
        });
        
        // Verify form fields are populated
        expect(screen.getByDisplayValue('existing value')).toBeInTheDocument();
        expect(screen.getByDisplayValue('42')).toBeInTheDocument();
    });
    
    test('saves step data and proceeds to next step', async () => {
        fetch
            .mockResolvedValueOnce({ ok: false }) // Load data (404 - no existing data)
            .mockResolvedValueOnce({
                ok: true,
                json: async () => ({ id: 1, flow_id: 1 })
            }); // Save data
        
        render(<MockedStep1Component />);
        
        // Fill form
        fireEvent.change(screen.getByLabelText(/field 1/i), {
            target: { value: 'test value' }
        });
        fireEvent.change(screen.getByLabelText(/field 2/i), {
            target: { value: '123' }
        });
        
        // Click next
        fireEvent.click(screen.getByText('Next'));
        
        // Verify save API call
        await waitFor(() => {
            expect(fetch).toHaveBeenCalledWith('/api/step1/', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    field1: 'test value',
                    field2: 123,
                    field3: false,
                    flow_id: 1
                })
            });
        });
        
        // Verify next step is called
        expect(mockUseWizard.nextStep).toHaveBeenCalled();
    });
    
    test('validates required fields', async () => {
        fetch.mockResolvedValueOnce({ ok: false }); // Load data (404)
        
        render(<MockedStep1Component />);
        
        // Try to proceed without filling required field
        fireEvent.click(screen.getByText('Next'));
        
        // Verify validation error
        expect(await screen.findByText(/field 1 is required/i)).toBeInTheDocument();
        expect(mockUseWizard.nextStep).not.toHaveBeenCalled();
    });
});
```

#### Backend Service Testing - Flow API Testing
```python
import pytest
from fastapi.testclient import TestClient
from main import app
from tests.test_utils import TestDatabase

client = TestClient(app)

class TestFlowOrchestration:
    """Comprehensive test suite for flow orchestration"""
    
    def setup_method(self):
        """Setup test database and data"""
        self.test_db = TestDatabase()
        self.flow_init_data = {
            "flow_type": "test_flow",
            "total_steps": 3,
            "user_id": "test_user"
        }
    
    def teardown_method(self):
        """Cleanup test data"""
        self.test_db.cleanup()
    
    def test_initialize_flow_empty_request(self):
        """Test flow initialization with empty request to prevent 422 errors"""
        response = client.post("/api/flow/initialize", json={})
        assert response.status_code == 422
    
    def test_initialize_flow_success(self):
        """Test successful flow initialization"""
        response = client.post("/api/flow/initialize", json=self.flow_init_data)
        assert response.status_code == 201
        
        data = response.json()
        assert data["id"] is not None
        assert data["flow_type"] == "test_flow"
        assert data["current_step"] == 1
        assert data["flow_status"] == "in_progress"
    
    def test_get_flow_state(self):
        """Test flow state retrieval"""
        # Initialize flow first
        init_response = client.post("/api/flow/initialize", json=self.flow_init_data)
        flow_id = init_response.json()["id"]
        
        # Get flow state
        response = client.get(f"/api/flow/{flow_id}")
        assert response.status_code == 200
        
        data = response.json()
        assert data["id"] == flow_id
        assert data["current_step"] == 1
    
    def test_update_flow_step(self):
        """Test flow step progression"""
        # Initialize flow
        init_response = client.post("/api/flow/initialize", json=self.flow_init_data)
        flow_id = init_response.json()["id"]
        
        # Update to step 2
        update_response = client.put(f"/api/flow/{flow_id}/step", json={
            "current_step": 2
        })
        assert update_response.status_code == 200
        assert update_response.json()["current_step"] == 2
    
    def test_consolidate_flow_data(self):
        """Test flow completion and data consolidation"""
        # Initialize flow
        init_response = client.post("/api/flow/initialize", json=self.flow_init_data)
        flow_id = init_response.json()["id"]
        
        # Add some step data (mock data for testing)
        step1_data = {
            "flow_id": flow_id,
            "field1": "test value",
            "field2": 123,
            "field3": True
        }
        client.post("/api/step1/", json=step1_data)
        
        # Consolidate flow
        consolidate_response = client.post(f"/api/flow/{flow_id}/consolidate")
        assert consolidate_response.status_code == 200
        
        result = consolidate_response.json()
        assert result["status"] == "success"
        assert "result" in result

class TestStep1Service:
    """Test step 1 service functionality"""
    
    def setup_method(self):
        """Setup test database"""
        self.test_db = TestDatabase()
        # Initialize a test flow
        flow_response = client.post("/api/flow/initialize", json={
            "flow_type": "test_flow",
            "total_steps": 3,
            "user_id": "test_user"
        })
        self.flow_id = flow_response.json()["id"]
    
    def teardown_method(self):
        """Cleanup test data"""
        self.test_db.cleanup()
    
    def test_save_step1_data_success(self):
        """Test successful step 1 data save"""
        step_data = {
            "flow_id": self.flow_id,
            "field1": "test value",
            "field2": 123,
            "field3": True
        }
        
        response = client.post("/api/step1/", json=step_data)
        assert response.status_code == 201
        
        data = response.json()
        assert data["flow_id"] == self.flow_id
        assert data["field1"] == "test value"
        assert data["field2"] == 123
    
    def test_step1_data_validation(self):
        """Test step 1 data validation"""
        invalid_data = {
            "flow_id": self.flow_id,
            "field1": "",  # Required field empty
            "field2": -1,  # Negative value
            "field3": True
        }
        
        response = client.post("/api/step1/", json=invalid_data)
        assert response.status_code == 400
        assert "Field 1 is required" in response.json()["detail"]
    
    def test_get_step1_data(self):
        """Test step 1 data retrieval"""
        # Save data first
        step_data = {
            "flow_id": self.flow_id,
            "field1": "test value",
            "field2": 123,
            "field3": True
        }
        client.post("/api/step1/", json=step_data)
        
        # Retrieve data
        response = client.get(f"/api/step1/{self.flow_id}")
        assert response.status_code == 200
        
        data = response.json()
        assert data["field1"] == "test value"
        assert data["field2"] == 123
    
    def test_update_existing_step1_data(self):
        """Test updating existing step 1 data"""
        # Save initial data
        initial_data = {
            "flow_id": self.flow_id,
            "field1": "initial value",
            "field2": 100,
            "field3": False
        }
        client.post("/api/step1/", json=initial_data)
        
        # Update data
        updated_data = {
            "flow_id": self.flow_id,
            "field1": "updated value",
            "field2": 200,
            "field3": True
        }
        response = client.post("/api/step1/", json=updated_data)
        assert response.status_code == 201
        
        # Verify updated values
        data = response.json()
        assert data["field1"] == "updated value"
        assert data["field2"] == 200
        assert data["field3"] == True
```

---

## Quality Assurance Framework - Flow Standards Compliance

### Validation Checklist - Flow Requirements Verification
Verify generated flows meet all requirements:

#### Flow Structure with react-use-wizard Validation
- ✅ Flow container component created using react-use-wizard Wizard wrapper
- ✅ Individual step components use useWizard hooks for navigation
- ✅ Flow container handles flow initialization and passes flowId to steps
- ✅ Each step component manages its own API integration to dedicated service
- ✅ Flow registered in left-navigation.json with appropriate showInNavigation setting

#### Backend Architecture Validation
- ✅ Flow control model created for flow state management
- ✅ Step-specific models created for normalized data storage
- ✅ Dedicated service created for each screen/step
- ✅ Flow service created for initialization, coordination, and consolidation
- ✅ All services registered in main.py and models in TableManager
- ✅ DataAccessController integration with .model_dump() conversion in all services

#### Step Components with react-use-wizard Validation
- ✅ Step components use useWizard hooks (nextStep, previousStep, activeStep, isFirstStep, isLastStep)
- ✅ Each step manages its own state, validation, and API calls
- ✅ Step components handle their own data persistence through dedicated services
- ✅ Step navigation controlled by react-use-wizard, not custom logic
- ✅ Flow data persisted to backend after each step completion

#### Testing and Integration Validation
- ✅ Flow initialization tested with empty requests to prevent 422 errors
- ✅ Each step service tested independently with FastAPI TestClient
- ✅ React-use-wizard navigation tested across all steps
- ✅ Data persistence validated across normalized step tables
- ✅ Flow functionality verified when screens are accessed
- ✅ Flow completion and data consolidation tested end-to-end

#### Error Handling and User Experience Validation
- ✅ Comprehensive error handling at both frontend and backend layers
- ✅ Loading states properly managed during API operations
- ✅ Form validation prevents invalid data submission
- ✅ Save and continue later functionality implemented
- ✅ Flow resumption capability for incomplete flows

### Quality Standards - Framework Standards Compliance
Ensure all generated flows meet framework standards:
- Complete implementations with no placeholder code
- Proper error handling with user-friendly messages and technical debug information
- Responsive design that works across different screen sizes
- Accessibility compliance with WCAG guidelines for multi-step forms
- Performance optimization for step transitions and data persistence
- Security considerations for flow state management and data validation

<!-- COMPLETION VERIFICATION - Flow-Specific Success Criteria -->
Before marking flow orchestration complete, verify appropriate mode completion:

**For BUILD-FLOW mode:**
☐ Complete flow container implemented with react-use-wizard Wizard wrapper
☐ All step components use useWizard hooks for proper navigation control
☐ Backend flow control and step-specific models created with normalized architecture
☐ Dedicated service implemented for each step with comprehensive validation
☐ Flow orchestration service handles initialization, coordination, and consolidation
☐ All services properly registered in main.py and models in TableManager
☐ Flow navigation and state management functional across all steps
☐ Data persistence working correctly for each step through dedicated services
☐ Comprehensive testing covers flow initialization, step progression, and completion
☐ Flow registered in left-navigation.json with appropriate visibility settings

**For EXTEND-FLOW mode:**
☐ New steps successfully integrated into existing react-use-wizard flow structure
☐ Enhanced functionality works seamlessly with existing step components
☐ New step services follow established patterns and maintain compatibility
☐ Existing flow functionality continues to work without regression
☐ Flow orchestration maintains proper step progression with additional steps
☐ Data consolidation updated to handle new step data and relationships
☐ Navigation integration properly handles extended step count
☐ Test coverage extended to include new steps and enhanced functionality
☐ Performance characteristics maintained or improved with flow extensions
☐ Documentation updated to reflect new step requirements and functionality

**For MERGE-STEPS mode:**
☐ All widget flow requirements implemented in fresh react-use-wizard architecture
☐ Existing valuable custom logic successfully integrated into new flow structure
☐ Previous flow functionality preserved where not superseded by new requirements
☐ Fresh flow architecture follows updated widget specifications exactly
☐ Backend services regenerated with enhanced data persistence and validation
☐ Custom step logic and validation patterns preserved where beneficial
☐ Flow orchestration maintains compatibility with existing navigation patterns
☐ Performance characteristics maintained or improved compared to previous implementation
☐ Migration strategy planned for transitioning to regenerated flow
☐ Comprehensive testing validates all regenerated flow functionality

**Universal verification for all modes:**
☐ Flow components follow React and react-use-wizard best practices
☐ Backend API endpoints properly documented with request/response examples
☐ Database operations optimized with proper DataAccessController usage
☐ Error handling provides sufficient detail for debugging and user guidance
☐ Widget flow requirements accurately implemented throughout full stack
☐ Security measures appropriate for flow operations and data handling
☐ Code follows established project patterns and naming conventions
☐ Integration with existing frontend and backend infrastructure functions correctly
☐ Flow is functional end-to-end with proper step transitions and data persistence
☐ Users can successfully complete flows and resume interrupted flows
<!-- END VERIFICATION -->

Execute appropriate flow orchestration mode based on user command context, ensuring comprehensive multi-step flow implementation using react-use-wizard while maintaining seamless integration with React + FastAPI + SQLAlchemy + DataAccessController technology stack.
