# fcfs
        FCFS algorithm in Python for HPC PAC1
        
        from tabulate import tabulate
        
        def fcfs_scheduling(jobs, total_cpus):
            jobs_sorted = sorted(jobs, key=lambda x: x['arrival'])
            cpus = {i: [] for i in range(1, total_cpus + 1)}  # {CPU: [(start, end, job_id)]}
            cpu_available_time = {i: 0 for i in range(1, total_cpus + 1)}
            resource_usage = 0
            max_end_time = 0
        
            for job in jobs_sorted:
                required_cpus = job['cpus']
                arrival = job['arrival']
                runtime = job['runtime']
                
                # Find earliest available CPUs
                available_cpus = sorted(
                    [cpu for cpu, time in cpu_available_time.items() if time <= arrival],
                    key=lambda x: cpu_available_time[x]
                )[:required_cpus]
        
                # Start time is max(arrival_time, earliest CPU availability)
                start_time = max(arrival, max(cpu_available_time[cpu] for cpu in available_cpus))
                end_time = start_time + runtime
        
                # Assign job to CPUs
                for cpu in available_cpus:
                    cpus[cpu].append((start_time, end_time, job['id']))
                    cpu_available_time[cpu] = end_time
        
                resource_usage += job['cpus'] * runtime
                max_end_time = max(max_end_time, end_time)
        
            # Format schedule
            schedule = {}
            for cpu in cpus:
                schedule[cpu] = sorted(cpus[cpu], key=lambda x: x[0])
                schedule[cpu] = [(job_id, start, end) for (start, end, job_id) in schedule[cpu]]
            
            return schedule, resource_usage, max_end_time
        
        def print_schedule_table(schedule, total_cpus, max_time=15):
            grid = []
            for cpu in range(1, total_cpus + 1):
                cpu_jobs = schedule.get(cpu, [])
                time_slots = [' '] * max_time
                for job in cpu_jobs:
                    job_id, start, end = job
                    for t in range(start, end):
                        if t <= max_time:
                            time_slots[t - 1] = str(job_id)
                grid.append([f"CPU {cpu}"] + time_slots)
            
            headers = ["Time\\CPU#"] + [str(i) for i in range(1, max_time + 1)]
            print(tabulate(grid, headers=headers, tablefmt="grid"))
        
        jobs = [
            {'id': 1, 'arrival': 1, 'runtime': 2, 'cpus': 4},
            {'id': 2, 'arrival': 2, 'runtime': 3, 'cpus': 2},
            {'id': 3, 'arrival': 3, 'runtime': 5, 'cpus': 1},
            {'id': 4, 'arrival': 3, 'runtime': 2, 'cpus': 2},
            {'id': 5, 'arrival': 4, 'runtime': 1, 'cpus': 5},
            {'id': 6, 'arrival': 7, 'runtime': 4, 'cpus': 5},
            {'id': 7, 'arrival': 9, 'runtime': 4, 'cpus': 3},
            {'id': 8, 'arrival': 9, 'runtime': 2, 'cpus': 1},
            {'id': 9, 'arrival': 11, 'runtime': 3, 'cpus': 3},
            {'id': 10, 'arrival': 12, 'runtime': 1, 'cpus': 2},
        ]
        
        total_cpus = 6
        schedule, resource_usage, total_time = fcfs_scheduling(jobs, total_cpus)
        
        print("FCFS Schedule:")
        print_schedule_table(schedule, total_cpus)
        
        print(f"\nResource Utilization Formula:")
        print(f"Utilization = Total Node-Hours Used / (Total CPUs × Total Time)")
        print(f"             = {resource_usage} / ({total_cpus} × {total_time})")
        print(f"             = {resource_usage} / {total_cpus * total_time}")
        print(f"             = {(resource_usage / (total_cpus * total_time)) * 100:.2f}%")
