#!env python3
import sys
import os
import re

column_width = 20

created_re = re.compile("^<CreateLog> (.*) was created.*")
monitor_enter_re = re.compile("^<MonitorLog> (.*) enters .*state '(.*)'.]")
state_enter_re = re.compile("^<StateLog> (.*) enters state '(.*)'[.]")
monitor_exit_re = re.compile("^<MonitorLog> (.*) exits .*state '(.*)'.]")
state_exit_re = re.compile("^<StateLog> (.*) exits state '(.*)'[.]")
send_re = re.compile("^<SendLog> '(.*)' in state .* sent event '([^ ']*).* to (.*)([(][^)]*[)])")
dequeue_re = re.compile("^<DequeueLog> '(.*)' dequeued event '([^ ']*)[ '].*")
pop_re = re.compile("^<PopLog> '(.*)' popped with unhandled event '([^']*)'")
halt_re = re.compile("^<HaltLog> (.*) halted")
error_re = re.compile("^<ErrorLog> ([^.]*)([.].*)?")

black = "\x1b[40m"
red = "\x1b[41m"
green = "\x1b[42m"
yellow = "\x1b[43m"
blue = "\x1b[44m"
magenta = "\x1b[45m"
cyan = "\x1b[46m"
white = "\x1b[47m"

def get_machines(lines):
    machines = set()
    for line in lines:
        m = created_re.match(line)
        if m is not None:
            machine = m.group(1)
            if not machine.startswith("Plang.CSharpRuntime"):
                machines.add(machine)
            continue
        m = state_enter_re.match(line)
        if m is not None:
            machine = m.group(1)
            if not machine.startswith("Plang.CSharpRuntime"):
                machines.add(machine)
            continue
    return list(machines)

def print_header(machines):
    first = True
    for machine in machines:
        if not first:
            print("|", end='')
        first = False
        print(("%%-%d.%ds" % (column_width, column_width)) % machine, end='')
    print("")

def create_machine_map(machines):
    machine_map = {}
    for machine in machines:
        machine_map[machine] = { "state": "", "action": "", "color": black }
    return machine_map

def process_log_line(line, machine_map):
    for k in machine_map.keys():
        machine_map[k]["action"] = ""
        machine_map[k]["color"] = black

    m = monitor_enter_re.match(line)
    if m is not None:
        machine = m.group(1)
        state = m.group(2)
        machine_map[machine]["action"] = state
        machine_map[machine]["color"] = green
        machine_map[machine]["state"] = state
        return True

    m = state_enter_re.match(line)
    if m is not None:
        machine = m.group(1)
        state = m.group(2)
        machine_map[machine]["action"] = state
        machine_map[machine]["color"] = green
        machine_map[machine]["state"] = state
        return True

    m = monitor_exit_re.match(line)
    if m is not None:
        machine = m.group(1)
        state = m.group(2)
        machine_map[machine]["action"] = state
        machine_map[machine]["color"] = red
        machine_map[machine]["state"] = ""
        return True

    m = state_exit_re.match(line)
    if m is not None:
        machine = m.group(1)
        state = m.group(2)
        machine_map[machine]["action"] = state
        machine_map[machine]["color"] = red
        machine_map[machine]["state"] = ""
        return True

    m = send_re.match(line)
    if m is not None:
        machine = m.group(1)
        event = m.group(2)
        machine_num = m.group(4)
        machine_map[machine]["action"] = event+"-->"+machine_num
        machine_map[machine]["color"] = blue
        return True

    m = dequeue_re.match(line)
    if m is not None:
        machine = m.group(1)
        event = m.group(2)
        machine_map[machine]["action"] = "-->"+event
        machine_map[machine]["color"] = magenta
        return True
    elif line.startswith("<DequeueLog>"):
        print("Failed to match: "+line)

    m = pop_re.match(line)
    if m is not None:
        machine = m.group(1)
        event = m.group(2)
        machine_map[machine]["action"] = "<<<"+event+">>>"
        machine_map[machine]["color"] = yellow
        return True

    m = halt_re.match(line)
    if m is not None:
        machine = m.group(1)
        machine_map[machine]["action"] = "!!! Halted !!!"
        machine_map[machine]["color"] = red
        machine_map[machine]["state"] = "dead"
        return True

    m = error_re.match(line)
    if m is not None:
        print("%s%s%s" % (red, line, black))
        return False

    return False

filenames = []
pos = 1
while pos < len(sys.argv):
    if sys.argv[pos].startswith("--column-width"):
        if sys.argv[pos].startswith("--column-width="):
            column_width = int(sys.argv[pos][15:])
            pos = pos + 1
        elif pos < len(sys.argv)-1:
            column_width = int(sys.argv[pos+1])
            pos = pos + 2
        else:
            print("Column with is specified as --column-width=nn  or --column-width nn")
            quit()
    else:
        filenames.append(sys.argv[pos])
        pos = pos + 1

for filename in filenames:
    file_in = open(filename, "r")
    log_lines = []
    for line in file_in:
        log_lines.append(line.strip())

    machines = get_machines(log_lines)
    print_header(machines)

    machine_map = create_machine_map(machines)

    lines_processed = 0
    for line in log_lines:
        if process_log_line(line, machine_map):
            lines_processed = lines_processed + 1

            if lines_processed >= 20:
                print_header(machines)
                lines_processed = 0

            first = True
            for machine in machines:
                if not first:
                    print("|",end="")
                first = False

                if machine_map[machine]["action"] != "":
                    print(("%%s%%-%d.%ds%%s" % (column_width, column_width)) % (machine_map[machine]["color"], machine_map[machine]["action"], black), end='')
                else:
                    print(("%%s%%-%d.%ds"  % (column_width, column_width)) % (black, machine_map[machine]["state"]), end='')
            print("")
    file_in.close()
