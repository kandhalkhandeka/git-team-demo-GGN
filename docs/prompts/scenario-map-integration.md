# Workflow Orchestration Integration

## Purpose
Automatically analyze project orchestration maps for multi-workflow relationships and integrate transition logic when related workflows are implemented in the codebase.

## Description
This prompt handles the intelligent integration of workflow orchestration maps that define relationships between multiple workflows in a project. It analyzes structure patterns, identifies implementation gaps, and generates seamless transition logic to create unified user experiences across related business processes.

---

## integrate-orchestration-map

### Context
When multiple workflows exist in a project with defined relationships and transition conditions, users need seamless integration between workflows without exposing internal orchestration concepts.

### Requirements Analysis
Before executing this prompt, ensure:
- scenario_map.json file exists in metadata directory
- Multiple workflows are present or planned in the project
- Related workflow implementations are available or completed
- Business domain context is established for appropriate terminology

### Auto-Detection Logic

**File Discovery:**
```javascript
// Discover scenario maps
const scenarioMapFiles = await file_search("**/scenario_map.json");

// Parse scenario map structure
const mapStructure = await read_file(scenarioMapFile);
const relationships = mapStructure.connections || mapStructure.relationships;
```

**Relationship Analysis:**
```javascript
// Extract workflow relationships
const workflowConnections = relationships.map(connection => ({
  from: connection.source || connection.from,
  to: connection.target || connection.to,
  conditions: connection.conditions || connection.triggers,
  dataFlow: connection.data_passing || connection.context
}));

// Identify current workflow context
const currentWorkflow = extractWorkflowFromUserRequest(userInput);
const relatedWorkflows = findRelatedWorkflows(currentWorkflow, workflowConnections);
```

**Implementation Assessment:**
```javascript
// Check project tracker for workflow completion
const implementedWorkflows = project_tracker.tasks
  .filter(task => task.status === "completed" && task.type === "workflow")
  .map(task => task.workflowId);

// Identify integration opportunities
const readyForIntegration = relatedWorkflows.filter(workflow => 
  implementedWorkflows.includes(workflow.id)
);
```

### Execution Process

**Phase 1: Orchestration Discovery**
1. **File Detection**: Automatically scan for scenario_map.json files using pattern matching
2. **Structure Analysis**: Parse scenario map structure to understand workflow relationships and data flows
3. **Context Extraction**: Identify business domain terminology and process naming conventions
4. **Relationship Mapping**: Extract connection patterns, conditions, and transition requirements

**Phase 2: Implementation Assessment**
1. **Codebase Analysis**: Use semantic search to identify existing workflow implementations
2. **Gap Identification**: Compare orchestration requirements with current implementation status
3. **Dependency Validation**: Verify prerequisite workflows are complete and functional
4. **Integration Readiness**: Determine which relationships are ready for implementation

**Phase 3: Transition Logic Generation**
1. **Business Rule Translation**: Convert scenario map conditions into domain-specific business rules
2. **Navigation Logic**: Generate conditional navigation based on business state rather than technical triggers
3. **Data Flow Implementation**: Create context passing mechanisms between related workflows
4. **User Experience Integration**: Build seamless transitions that feel natural to business users

**Phase 4: Domain Translation**
1. **Terminology Mapping**: Replace scenario map technical terms with business domain language
2. **Process Naming**: Use domain-appropriate names for workflows and transitions
3. **User Interface Integration**: Generate business-focused navigation and progress indicators
4. **Documentation Generation**: Create user-friendly process descriptions and help content

### Implementation Patterns

**Conditional Navigation Pattern:**
```javascript
// Generated transition logic (domain-specific)
const handleProcessCompletion = async (processData) => {
  // Business rule evaluation (not technical conditions)
  if (processData.requiresFollowup && processData.followupType === 'assessment') {
    await saveProcessContext(processData);
    navigateToAssessmentWorkflow(processData.contextId);
  } else if (processData.status === 'complete') {
    showCompletionSummary(processData);
  }
};

// Cross-workflow data context
const WorkflowContext = {
  passDataToNextProcess: (workflowData, targetProcess) => {
    const contextData = extractRelevantContext(workflowData);
    sessionStorage.setItem(`${targetProcess}_context`, JSON.stringify(contextData));
  },
  
  loadPreviousContext: (sourceProcess) => {
    const contextKey = `${sourceProcess}_context`;
    return JSON.parse(sessionStorage.getItem(contextKey) || '{}');
  }
};
```

**Business Process Integration Pattern:**
```python
# Generated service integration (domain-focused)
class ProcessOrchestrationService:
    def __init__(self, db_session):
        self.db = db_session
        
    async def complete_process_step(self, process_id: str, step_data: dict):
        """Complete current process step and determine next actions"""
        
        # Update current process state
        process = await self.update_process_status(process_id, step_data)
        
        # Evaluate business rules for next steps
        next_actions = await self.evaluate_next_actions(process)
        
        # Handle workflow transitions based on business logic
        if next_actions.requires_additional_process:
            return await self.initiate_related_process(process, next_actions)
        
        return process
        
    async def evaluate_next_actions(self, process):
        """Determine next steps based on business rules"""
        # Business rule evaluation using domain terminology
        # No technical orchestration concepts exposed
        pass
```

**Unified User Experience Pattern:**
```javascript
// Generated user interface (business-focused)
const ProcessNavigation = () => {
  const { currentProcess, availableNextSteps } = useProcessState();
  
  return (
    <div className="process-navigation">
      <ProcessProgress current={currentProcess} />
      
      {availableNextSteps.map(step => (
        <NavigationButton 
          key={step.id}
          label={step.businessLabel}  // Domain-specific labels
          description={step.businessDescription}
          onClick={() => handleProcessTransition(step)}
        />
      ))}
    </div>
  );
};

const useProcessState = () => {
  // Business-focused state management
  // Technical orchestration logic abstracted away
};
```

### Quality Assurance Standards

**Domain Abstraction Requirements:**
- All generated code uses business domain terminology exclusively
- Technical scenario map concepts never appear in user-facing components
- Workflow relationships expressed as business process flows
- Navigation and transitions feel natural to domain experts

**Integration Validation:**
- Cross-workflow data flow maintains consistency and integrity
- Business rule evaluation produces expected outcomes
- User experience flows are intuitive and domain-appropriate
- Performance meets requirements for production use

**Testing Patterns:**
```javascript
// Generated test suites focus on business scenarios
describe('Business Process Integration', () => {
  test('should transition to assessment when follow-up required', async () => {
    const processData = createSampleProcessData({ requiresFollowup: true });
    const result = await processOrchestration.completeProcess(processData);
    
    expect(result.nextProcess).toBe('assessment');
    expect(result.contextPassed).toBeTruthy();
  });
  
  test('should maintain business context across workflow transitions', async () => {
    // Test domain-specific context preservation
    // No technical orchestration testing exposed
  });
});
```

### Success Criteria

**Technical Integration:**
- Scenario map logic executes automatically without user intervention
- Transition conditions evaluate correctly based on business rules
- Cross-workflow data flow maintains consistency
- Generated code follows established architecture patterns

**User Experience:**
- Multi-workflow processes feel like unified business flows
- Navigation and transitions are intuitive and business-focused
- No technical metadata concepts visible in user interfaces
- Process orchestration enhances productivity without complexity

**Business Value:**
- Workflow integration reduces manual coordination effort
- Business users can complete related processes seamlessly
- Domain expertise is preserved in generated terminology
- Process efficiency improves through intelligent scenario map orchestration

### Integration Points

**Coordinates with other prompts:**
- `flow-orchestration.md`: Enhances multi-step flows with cross-workflow transitions
- `rules-engine.md`: Implements business rules for orchestration decisions
- `screen-generation.md`: Generates navigation components for workflow transitions
- `domain-model.md`: Creates shared data models for cross-workflow context

**Project tracker integration:**
- Automatically adds scenario map integration tasks when multiple workflows detected
- Manages dependencies between workflow implementations
- Tracks integration completion and validation status
- Updates progress indicators for multi-workflow projects

### Example Task Generation

When scenario_map.json is detected and workflows are ready for integration:

```json
{
  "taskId": "orchestration_integration_001",
  "title": "Integrate Multi-Workflow Business Process Flow",
  "promptFile": "scenario-map-integration.md",
  "mode": "integrate-orchestration-map",
  "autoExecute": true,
  "dependencies": ["workflow_001_complete", "workflow_002_complete"],
  "deliverables": [
    "Cross-workflow navigation logic",
    "Business process integration services",
    "Conditional transition implementation",
    "Domain-specific user experience flows"
  ],
  "validationSteps": [
    "Business process flow testing",
    "Cross-workflow data integrity validation",
    "User experience continuity verification",
    "Domain terminology accuracy check"
  ]
}
```

This approach ensures that scenario map intelligence enhances the user experience while maintaining complete domain abstraction and business-focused terminology throughout the generated application.
