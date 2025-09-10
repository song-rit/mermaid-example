```mermaid
flowchart TD
    A[Start: Code Commit] --> B{Is Branch Protected?}
    
    B -->|Yes| C[Create Pull Request]
    B -->|No| D[Direct Push to Branch]
    
    C --> E{PR Review Status}
    E -->|Needs Changes| F[Developer Updates Code]
    F --> C
    E -->|Approved| G[Merge PR]
    
    D --> H[Trigger CI Pipeline]
    G --> H
    
    H --> I[Run Unit Tests]
    I --> J{Tests Pass?}
    
    J -->|No| K[Send Failure Notification]
    K --> L[Developer Fixes Issues]
    L --> A
    
    J -->|Yes| M[Build Application]
    M --> N{Build Successful?}
    
    N -->|No| O[Log Build Errors]
    O --> K
    
    N -->|Yes| P[Run Integration Tests]
    P --> Q{Integration Tests Pass?}
    
    Q -->|No| R[Analyze Test Failures]
    R --> K
    
    Q -->|Yes| S[Security Scan]
    S --> T{Security Issues Found?}
    
    T -->|Yes| U[Generate Security Report]
    U --> V{Critical Vulnerabilities?}
    V -->|Yes| K
    V -->|No| W[Create Security Ticket]
    W --> X[Deploy to Staging]
    
    T -->|No| X
    
    X --> Y[Run Smoke Tests]
    Y --> Z{Smoke Tests Pass?}
    
    Z -->|No| AA[Rollback Staging]
    AA --> K
    
    Z -->|Yes| BB{Deploy to Production?}
    BB -->|No| CC[Manual Testing in Staging]
    CC --> DD[QA Approval]
    DD --> BB
    
    BB -->|Yes| EE[Blue-Green Deployment]
    EE --> FF[Deploy to Blue Environment]
    FF --> GG[Health Check Blue]
    GG --> HH{Blue Environment Healthy?}
    
    HH -->|No| II[Keep Green Active]
    II --> JJ[Investigate Blue Issues]
    JJ --> K
    
    HH -->|Yes| KK[Switch Traffic to Blue]
    KK --> LL[Monitor Metrics]
    LL --> MM{Performance OK?}
    
    MM -->|No| NN[Rollback to Green]
    NN --> OO[Incident Response]
    OO --> K
    
    MM -->|Yes| PP[Gradual Traffic Increase]
    PP --> QQ[100% Traffic to Blue]
    QQ --> RR[Decommission Green]
    RR --> SS[Update DNS Records]
    SS --> TT[Send Success Notification]
    TT --> UU[Update Documentation]
    UU --> VV[End: Deployment Complete]
    
    %% Parallel processes
    X --> WW[Update Load Balancer Config]
    X --> XX[Backup Database]
    WW --> Y
    XX --> Y
    
    %% Styling
    classDef startEnd fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef decision fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef process fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef error fill:#ffebee,stroke:#b71c1c,stroke-width:2px
    classDef success fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    
    class A,VV startEnd
    class B,E,J,N,Q,T,V,Z,BB,HH,MM decision
    class C,D,G,H,I,M,P,S,X,Y,EE,FF,GG,KK,LL,PP,QQ,RR,SS,UU,WW,XX,CC,DD process
    class K,L,O,R,AA,II,JJ,NN,OO error
    class TT,VV success