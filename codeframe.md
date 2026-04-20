graph LR
    %% 样式定义
    classDef master fill:#f9f2f4,stroke:#d08770,stroke-width:2px;
    classDef controller fill:#e5f5fd,stroke:#5e81ac,stroke-width:2px;
    classDef component fill:#ebf5e7,stroke:#a3be8c,stroke-width:2px;
    classDef device fill:#fdf1ea,stroke:#d08770,stroke-width:2px;
    classDef bsp fill:#eceff4,stroke:#4c566a,stroke-width:2px;

    subgraph MasterNode [1. Master Node]
        Pi5[Raspberry Pi 5<br>Vision & Control Logic]:::master
    end

    subgraph ControllerLayer [2. Controller Layer]
        Main[Main Loop<br>Timeout & Monitor]:::controller
        ISR[500Hz Timer ISR<br>Hard Real-Time]:::controller
    end

    subgraph ComponentLayer [3. Component Layer]
        Parser[UART_Parser<br>FSM Protocol]:::component
        Math[FastMath<br>Sine LUT]:::component
        Motor[GimbalMotor Class<br>Pitch & Yaw]:::component
    end

    subgraph DeviceLayer [4. Device / Library Layer]
        FOC[SimpleFOC<br>BLDCDriver3PWM]:::device
    end

    subgraph BSPLayer [5. BSP / Hardware]
        HW_Timer((Hardware<br>Timer)):::bsp
        HW_UART((UART0<br>RX/TX)):::bsp
        HW_PWM((GPIO / PWM<br>Pins)):::bsp
    end

    %% 数据流连线
    Pi5 -- "14-Byte Packet" --> HW_UART
    HW_UART -- "RX IRQ" --> Parser
    Parser -- "target_pitch_vel<br>target_yaw_vel" -.-> Main
    Parser -- "Velocity Demand" --> Motor

    HW_Timer -- "2ms Interrupt" --> ISR
    ISR -- "updateSPWM(dt)" --> Motor
    
    Motor -- "fast_sin(angle)" --> Math
    Motor -- "Phase U, V, W (Volts)" --> FOC
    FOC -- "Duty Cycle" --> HW_PWM
