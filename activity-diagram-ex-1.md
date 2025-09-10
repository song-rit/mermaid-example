```mermaid
flowchart TD
    Start([Start]) --> GetInput[Get User Input]
    GetInput --> ValidateInput{Valid Input?}
    
    ValidateInput -->|No| ShowError[Show Error Message]
    ShowError --> GetInput
    
    ValidateInput -->|Yes| ProcessData[Process Data]
    ProcessData --> CheckCondition{Check Condition}
    
    CheckCondition -->|Condition A| PathA[Execute Path A]
    CheckCondition -->|Condition B| PathB[Execute Path B]
    CheckCondition -->|Default| PathDefault[Execute Default Path]
    
    PathA --> MergePoint[Merge Results]
    PathB --> MergePoint
    PathDefault --> MergePoint
    
    MergePoint --> SaveData[Save Data to Database]
    SaveData --> Success{Save Successful?}
    
    Success -->|Yes| SendNotification[Send Success Notification]
    Success -->|No| HandleError[Handle Save Error]
    
    SendNotification --> End([End])
    HandleError --> Retry{Retry?}
    
    Retry -->|Yes| SaveData
    Retry -->|No| LogError[Log Error]
    LogError --> End
    
    %% Styling
    classDef startEnd fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef decision fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef error fill:#ffebee,stroke:#c62828,stroke-width:2px
    
    class Start,End startEnd
    class GetInput,ProcessData,PathA,PathB,PathDefault,MergePoint,SaveData,SendNotification process
    class ValidateInput,CheckCondition,Success,Retry decision
    class ShowError,HandleError,LogError error

    click C7 "https://example.com/track/ORDER123" "Open tracking"
