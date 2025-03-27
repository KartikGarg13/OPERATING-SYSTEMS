# OPERATING-SYSTEMS
DEADLOCK PREVENTION AND RECOVERY TOOLKIT

from collections import deque

import numpy as np
import matplotlib.pyplot as plt

def displayRAG(allocation, request, available):
    print("\nResource Allocation Graph (RAG):")
    for i in range(len(allocation)):
        print(f"Process P{i}: Holds {allocation[i]} | Requests {request[i]}")
    print(f"Available Resources: {available}")
    
    labels = []
    sizes = []
    colors = ['blue', 'orange', 'green', 'red', 'purple', 'brown']
    
    for i in range(len(allocation)):
        labels.append(f'P{i} Allocated')
        sizes.append(sum(allocation[i]))
    
    labels.append('Available')
    sizes.append(sum(available))
    
    plt.figure(figsize=(7, 7))
    plt.pie(sizes, labels=labels, autopct='%1.1f%%', colors=colors[:len(labels)], startangle=140)
    plt.title("Resource Allocation Pie Chart")
    plt.show()

def isSafeState(allocation, need, available):
    work = available[:]
    finish = [False] * len(allocation)
    safe_sequence = []
    
    while True:
        allocated = False
        for i in range(len(allocation)):
            if not finish[i] and all(need[i][j] <= work[j] for j in range(len(available))):
                work = [work[j] + allocation[i][j] for j in range(len(available))]
                finish[i] = True
                safe_sequence.append(i)
                allocated = True
        
        if not allocated:
            break
    
    return all(finish), safe_sequence if all(finish) else []

def canAllocate(process, request, allocation, need, available):
    if any(request[j] > available[j] or request[j] > need[process][j] for j in range(len(available))):
        return False  
    
    temp_available = [available[j] - request[j] for j in range(len(available))]
    temp_allocation = [allocation[i][:] for i in range(len(allocation))]
    temp_need = [need[i][:] for i in range(len(need))]
    
    for j in range(len(available)):
        temp_allocation[process][j] += request[j]
        temp_need[process][j] -= request[j]
    
    return isSafeState(temp_allocation, temp_need, temp_available)[0]

def allocateResource(process, request, allocation, need, available):
    if canAllocate(process, request, allocation, need, available):
        for j in range(len(available)):
            available[j] -= request[j]
            allocation[process][j] += request[j]
            need[process][j] -= request[j]
        print(f"Request granted for P{process}.")
    else:
        print(f"Request denied for P{process} (would cause deadlock).")

def detectDeadlock(allocation, request, available):
    return [i for i in range(len(allocation)) if not isSafeState(allocation, [[max(0, need) for need in row] for row in request], available)[0]]

def recoverFromDeadlock(allocation, request, available, need):
    deadlocked_processes = detectDeadlock(allocation, request, available)
    
    if not deadlocked_processes:
        print("No deadlock detected.")
        return
    
    print(f"Deadlock detected! Processes involved: {deadlocked_processes}")
    
    while deadlocked_processes:
        victim = deadlocked_processes.pop(0)
        print(f"Preempting resources from P{victim}...")
        
        for j in range(len(available)):
            available[j] += allocation[victim][j]
            allocation[victim][j] = 0
            request[victim][j] = 0
            need[victim][j] = 0
        
        deadlocked_processes = detectDeadlock(allocation, request, available)
    
    print("Deadlock resolved!")

# Dynamic User Input for Deadlock Simulation
processes = int(input("Enter the number of processes: "))
resources = int(input("Enter the number of resources: "))

allocation = []
request = []
max_need = []

print("Enter allocation matrix:")
for i in range(processes):
    while True:
        row = list(map(int, input(f"P{i}: ").split()))
        if len(row) == resources:
            allocation.append(row)
            break
        else:
            print(f"Error: Expected {resources} values, got {len(row)}. Try again.")

print("Enter request matrix:")
for i in range(processes):
    while True:
        row = list(map(int, input(f"P{i}: ").split()))
        if len(row) == resources:
            request.append(row)
            break
        else:
            print(f"Error: Expected {resources} values, got {len(row)}. Try again.")

print("Enter max need matrix:")
for i in range(processes):
    while True:
        row = list(map(int, input(f"P{i}: ").split()))
        if len(row) == resources:
            max_need.append(row)
            break
        else:
            print(f"Error: Expected {resources} values, got {len(row)}. Try again.")

while True:
    available = list(map(int, input("Enter available resources: ").split()))
    if len(available) == resources:
        break
    else:
        print(f"Error: Expected {resources} values, got {len(available)}. Try again.")

# Compute Need Matrix Safely
need = [[max_need[i][j] - allocation[i][j] for j in range(resources)] for i in range(processes)]

displayRAG(allocation, request, available)

while True:
    action = input("Enter action (request/release/display/exit): ")
    if action == "request":
        p = int(input("Enter process ID: "))
        req = list(map(int, input("Enter request vector: ").split()))
        allocateResource(p, req, allocation, need, available)
    elif action == "release":
        p = int(input("Enter process ID to release resources: "))
        for j in range(resources):
            available[j] += allocation[p][j]
            allocation[p][j] = 0
            need[p][j] = max_need[p][j]
    elif action == "display":
        displayRAG(allocation, request, available)
    elif action == "exit":
        break
    else:
        print("Invalid action. Try again.")
    
    deadlocked_procs = detectDeadlock(allocation, request, available)
    if deadlocked_procs:
        recoverFromDeadlock(allocation, request, available, need)

displayRAG(allocation, request, available)
