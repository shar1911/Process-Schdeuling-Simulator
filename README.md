Features - 
1) User-Friendly Interface: The application features an intuitive interface where users can easily select a scheduling algorithm and add processes with specified arrival times, burst times, and priorities.
2)Multiple Scheduling Algorithms: Supports First-Come-First-Serve (FCFS), Shortest Job First (SJF), Priority Scheduling, Shortest Remaining Time Next (SRTN), and Round Robin.
3)Dynamic Process Management: Users can add multiple processes, which are dynamically displayed in a table. The scheduling algorithms are simulated in real-time.
4)Visual Progress Indicators: Each process's execution is visually indicated using progress bars, enhancing the understanding of the scheduling flow.
5)Detailed Logging: Provides real-time logging of process execution, including start and end times, turnaround times, and waiting times.
6)Performance Metrics: Calculates and displays average turnaround and waiting times for each algorithm, aiding in performance comparison and analysis.
Technical Details
1)Programming Language: Python
2)GUI Library: Tkinter
3)Concurrency: Utilizes threading to simulate process execution and maintain a responsive UI.
4)Process Queue: Implemented using Python's deque for efficient FIFO operations.
Usage
To use the CPU Scheduler Simulation:

1)Launch the application.
2)Select a scheduling algorithm from the dropdown menu.
3)Add processes by specifying their arrival times, burst times, and priorities (if applicable).
4)Click "Simulate" to run the selected scheduling algorithm.
5)Observe the process execution through the progress bars and review the detailed logs for performance metrics.
This project serves as an educational tool for students and professionals interested in operating systems and process management, providing practical insights into the behavior of different CPU scheduling strategies.






