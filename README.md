# Process-Schdeuling-Simulator
The CPU Scheduler Simulation project is a desktop application designed to simulate various CPU scheduling algorithms. Developed using Python's Tkinter library for the graphical user interface, this tool provides a hands-on approach to understanding how different scheduling algorithms manage processes in an operating system.

CODE
import tkinter as tk
from tkinter import ttk
from collections import deque
from tkinter import simpledialog
import threading
import time


class Process:

  def __init__(self, pid, arrival_time, burst_time, priority=None):
    self.pid = pid
    self.arrival_time = arrival_time
    self.burst_time = burst_time
    self.remaining_time = burst_time
    self.priority = priority


class SchedulerApp:

  def __init__(self, root):
    self.root = root
    self.root.title("CPU Scheduler Simulation")
    self.root.geometry("600x400")

    self.algorithm_var = tk.StringVar()
    self.processes = []
    self.process_queue = deque()

    self.create_widgets()

  def create_widgets(self):
    main_frame = ttk.Frame(self.root, padding="20")
    main_frame.grid(row=0, column=0)

    algorithm_label = ttk.Label(main_frame,
                                text="Select Scheduling Algorithm:")
    algorithm_label.grid(row=0, column=0, sticky="w")

    algorithm_combobox = ttk.Combobox(
        main_frame,
        textvariable=self.algorithm_var,
        values=["FCFS", "SJF", "Priority", "SRTN", "Round Robin"])
    algorithm_combobox.grid(row=0, column=1, padx=10)

    process_frame = ttk.LabelFrame(main_frame, text="Processes")
    process_frame.grid(row=1,
                       column=0,
                       columnspan=2,
                       padx=10,
                       pady=10,
                       sticky="w")

    self.process_table = ttk.Treeview(process_frame,
                                      columns=("pid", "arrival_time",
                                               "burst_time", "priority"))
    self.process_table.heading("#0", text="")
    self.process_table.heading("pid", text="PID")
    self.process_table.heading("arrival_time", text="Arrival Time")
    self.process_table.heading("burst_time", text="Burst Time")
    self.process_table.heading("priority", text="Priority")
    self.process_table.pack()

    add_process_button = ttk.Button(process_frame,
                                    text="Add Process",
                                    command=self.add_process)
    add_process_button.pack(pady=5)

    simulate_button = ttk.Button(main_frame,
                                 text="Simulate",
                                 command=self.simulate)
    simulate_button.grid(row=2, column=0, columnspan=2, pady=10)

    self.log_text = tk.Text(main_frame, height=10, width=50)
    self.log_text.grid(row=3, column=0, columnspan=2)

  def add_process(self):
    pid = len(self.processes) + 1
    arrival_time = simpledialog.askinteger(
        "Arrival Time", f"Enter arrival time for Process {pid}:")
    burst_time = simpledialog.askinteger(
        "Burst Time", f"Enter burst time for Process {pid}:")
    if self.algorithm_var.get() == "Priority":
      priority = simpledialog.askinteger("Priority",
                                         f"Enter priority for Process {pid}:")
      self.processes.append(Process(pid, arrival_time, burst_time, priority))
    else:
      self.processes.append(Process(pid, arrival_time, burst_time))

    self.update_process_table()

  def update_process_table(self):
    self.process_table.delete(*self.process_table.get_children())
    for process in self.processes:
      values = (process.pid, process.arrival_time, process.burst_time,
                process.priority if hasattr(process, 'priority') else "")
      self.process_table.insert("", "end", text="", values=values)

  def simulate(self):
    self.process_queue.clear()
    for process in self.processes:
      self.process_queue.append(process)

    algorithm = self.algorithm_var.get()
    self.log_text.delete(1.0, tk.END)

    simulation_thread = threading.Thread(target=self.run_simulation,
                                         args=(algorithm, ))
    simulation_thread.start()

  def log_message(self, message):
    self.log_text.insert(tk.END, message + "\n")
    self.log_text.see(tk.END)

  def run_simulation(self, algorithm):
    if algorithm == "FCFS":
      self.run_fcfs()
    elif algorithm == "SJF":
      self.run_sjf()
    elif algorithm == "Priority":
      self.run_priority()
    elif algorithm == "SRTN":
      self.run_srtn()
    elif algorithm == "Round Robin":
      self.run_round_robin()

  def run_fcfs(self):
    current_time = 0
    total_turnaround_time = 0
    total_waiting_time = 0

    # Sort the process queue based on arrival time
    self.process_queue = deque(sorted(self.process_queue, key=lambda x: x.arrival_time))

    while self.process_queue:
        current_process = self.process_queue.popleft()
        current_time = max(current_time, current_process.arrival_time)
        self.log_message(f"Process {current_process.pid} started execution.")
        self.update_progress(current_process, current_process.burst_time)
        completion_time = current_time + current_process.burst_time
        turnaround_time = completion_time - current_process.arrival_time
        waiting_time = turnaround_time - current_process.burst_time
        self.log_message(f"Process {current_process.pid} completed execution.")
        self.log_message(
            f"Turnaround Time: {turnaround_time}, Waiting Time: {waiting_time}")
        total_turnaround_time += turnaround_time
        total_waiting_time += waiting_time
        current_time = completion_time

    avg_turnaround_time = total_turnaround_time / len(self.processes)
    avg_waiting_time = total_waiting_time / len(self.processes)
    self.log_message(
        f"Average Turnaround Time: {avg_turnaround_time}, Average Waiting Time: {avg_waiting_time}"
    )

 
  def run_sjf(self):
    current_time = 0
    total_turnaround_time = 0
    total_waiting_time = 0

    while self.process_queue:
      arrived_processes = [
          p for p in self.process_queue if p.arrival_time <= current_time
      ]
      if not arrived_processes:
        current_time += 1
        continue
      current_process = min(arrived_processes, key=lambda x: x.burst_time)
      self.process_queue.remove(current_process)
      current_time = max(current_time, current_process.arrival_time)
      self.log_message(f"Process {current_process.pid} started execution.")
      self.update_progress(current_process, current_process.burst_time)
      completion_time = current_time + current_process.burst_time
      turnaround_time = completion_time - current_process.arrival_time
      waiting_time = turnaround_time - current_process.burst_time
      self.log_message(f"Process {current_process.pid} completed execution.")
      self.log_message(
          f"Turnaround Time: {turnaround_time}, Waiting Time: {waiting_time}")
      total_turnaround_time += turnaround_time
      total_waiting_time += waiting_time
      current_time = completion_time

    avg_turnaround_time = total_turnaround_time / len(self.processes)
    avg_waiting_time = total_waiting_time / len(self.processes)
    self.log_message(
        f"Average Turnaround Time: {avg_turnaround_time}, Average Waiting Time: {avg_waiting_time}"
    )

  def update_progress(self, process, burst_time):
    progress_frame = ttk.LabelFrame(self.root, text=f"Process {process.pid}")
    progress_frame.grid(row=len(self.processes) + 1,
                        column=0,
                        padx=10,
                        pady=10,
                        sticky="w")

    progress = ttk.Progressbar(progress_frame,
                               orient="horizontal",
                               length=200,
                               mode="determinate")
    progress.grid(row=0, column=0, padx=5, pady=5)

    progress["maximum"] = burst_time

    for i in range(burst_time + 1):
      progress["value"] = i
      progress.update_idletasks()
      time.sleep(0.1)

    progress.destroy()
    progress_frame.destroy()

  def run_priority(self):
    current_time = 0
    total_turnaround_time = 0
    total_waiting_time = 0

    while self.process_queue:
      arrived_processes = [
          p for p in self.process_queue if p.arrival_time <= current_time
      ]
      if not arrived_processes:
        current_time += 1
        continue
      current_process = min(arrived_processes,
                            key=lambda x: (x.priority, x.burst_time))
      self.process_queue.remove(current_process)
      current_time = max(current_time, current_process.arrival_time)
      self.log_message(f"Process {current_process.pid} started execution.")
      self.update_progress(current_process, current_process.burst_time)
      completion_time = current_time + current_process.burst_time
      turnaround_time = completion_time - current_process.arrival_time
      waiting_time = turnaround_time - current_process.burst_time
      self.log_message(f"Process {current_process.pid} completed execution.")
      self.log_message(
          f"Turnaround Time: {turnaround_time}, Waiting Time: {waiting_time}")
      total_turnaround_time += turnaround_time
      total_waiting_time += waiting_time
      current_time = completion_time

    avg_turnaround_time = total_turnaround_time / len(self.processes)
    avg_waiting_time = total_waiting_time / len(self.processes)
    self.log_message(
        f"Average Turnaround Time: {avg_turnaround_time}, Average Waiting Time: {avg_waiting_time}"
    )


  def run_round_robin(self):
    current_time = 0
    total_turnaround_time = 0
    total_waiting_time = 0
    time_quantum = 2

    # Sort the process queue based on arrival time
    self.process_queue = deque(sorted(self.process_queue, key=lambda x: x.arrival_time))

    while self.process_queue:
      current_process = self.process_queue.popleft()
      current_time = max(current_time, current_process.arrival_time)
      remaining_time = current_process.remaining_time

      if remaining_time <= time_quantum:
        self.update_progress(current_process, remaining_time)
        completion_time = current_time + remaining_time
        self.log_message(f"Process {current_process.pid} executing from time {current_time} and completes at time {completion_time}.")
        turnaround_time = completion_time - current_process.arrival_time
        waiting_time = turnaround_time - current_process.burst_time
        self.log_message(f"Process {current_process.pid} completed execution at time {completion_time}.")
        self.log_message(f"Turnaround Time: {turnaround_time}, Waiting Time: {waiting_time}")
        total_turnaround_time += turnaround_time
        total_waiting_time += waiting_time
        current_time = completion_time
      else:
        self.update_progress(current_process, time_quantum)
        completion_time = current_time + time_quantum
        self.log_message(f"Process {current_process.pid} executing from time {current_time} to time {completion_time} and gets preempted.")
        current_process.remaining_time -= time_quantum
        self.process_queue.append(current_process)
        current_time = completion_time

    avg_turnaround_time = total_turnaround_time / len(self.processes)
    avg_waiting_time = total_waiting_time / len(self.processes)
    self.log_message(f"Average Turnaround Time: {avg_turnaround_time}, Average Waiting Time: {avg_waiting_time}")
   


  def run_srtn(self):
        current_time = 0
        total_turnaround_time = 0
        total_waiting_time = 0
        n = len(self.processes)

        process_queue_copy = deque(self.process_queue)
        self.current_process = None  # Initialize current_process

        while process_queue_copy or self.current_process is not None:
            if self.current_process is None and process_queue_copy:
                arrived_processes = [p for p in process_queue_copy if p.arrival_time <= current_time]
                if arrived_processes:
                    self.current_process = min(arrived_processes, key=lambda x: x.remaining_time)
                    process_queue_copy.remove(self.current_process)

            if self.current_process is not None and process_queue_copy:
                arrived_processes = [p for p in process_queue_copy if p.arrival_time <= current_time]
                if arrived_processes:
                    self.next_current_process = min(arrived_processes, key=lambda x: x.remaining_time)
                    if self.next_current_process.remaining_time < self.current_process.remaining_time:
                        process_queue_copy.remove(self.next_current_process)
                        process_queue_copy.append(self.current_process)
                        self.current_process = self.next_current_process

            if self.current_process is not None:
                if self.current_process.remaining_time == 0:
                    completion_time = current_time
                    turnaround_time = completion_time - self.current_process.arrival_time
                    waiting_time = turnaround_time - self.current_process.burst_time
                    self.log_message(f"Process {self.current_process.pid} completed execution at time {completion_time}.")
                    self.log_message(f"Turnaround Time: {turnaround_time}, Waiting Time: {waiting_time}")
                    total_turnaround_time += turnaround_time
                    total_waiting_time += waiting_time
                    self.current_process = None
                    current_time -= 1
                else:
                    self.log_message(f"Process {self.current_process.pid} executing at time {current_time}.")
                    self.update_progress(self.current_process, 1)
                    self.current_process.remaining_time -= 1
               
            current_time += 1

        self.log_message(f"Total Turnaround Time: {total_turnaround_time}, Number of processes: {n}")
        avg_turnaround_time = total_turnaround_time / n
        avg_waiting_time = total_waiting_time / n
        self.log_message(f"Average Turnaround Time: {avg_turnaround_time}, Average Waiting Time: {avg_waiting_time}")



if __name__ == "__main__":
  root = tk.Tk()
  app = SchedulerApp(root)
  root.mainloop()
