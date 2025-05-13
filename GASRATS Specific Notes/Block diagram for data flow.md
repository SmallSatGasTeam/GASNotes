```mermaid
flowchart LR

    subgraph Data Producers
        A[Sensors]
        B[GPS Data]
        C[State Data]
        D[Camera Data]
    end
    
    subgraph Data Handlers
        E[ADCS Board]
        F[Data Collector]
        G[File Manager]
    end

    subgraph Data Packetizers
        H[Downlink]
        I[Transmission]
    end

    A --> E
    B --> E
    E --> F
    C --> F
    D --> G
    F --> H
    G --> H
    H --> I
```
