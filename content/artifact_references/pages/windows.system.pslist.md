---
title: Windows.System.Pslist
hidden: true
tags: [Client Artifact]
---

List processes and their running binaries.


<pre><code class="language-yaml">
name: Windows.System.Pslist
description: |
  List processes and their running binaries.

parameters:
  - name: ProcessRegex
    default: .
    type: regex
  - name: PidRegex
    default: .
    type: regex
  - name: ExePathRegex
    default: .
    type: regex
  - name: CommandLineRegex
    default: .
    type: regex
  - name: UsernameRegex
    default: .
    type: regex
  - name: UntrustedAuthenticode
    description: Show only Executables that are not trusted by Authenticode.
    type: bool
  - name: UseTracker
    type: bool
    description: If set we use the process tracker.

sources:
  - precondition: SELECT OS From info() where OS = 'windows'

    query: |
        LET ProcList = SELECT * FROM if(condition=UseTracker,
        then={
          SELECT Pid, Ppid, NULL AS TokenIsElevated,
                 Username, Name, CommandLine, Exe, NULL AS Memory
          FROM process_tracker_pslist()
        }, else={
          SELECT * FROM pslist()
        })

        SELECT Pid, Ppid, TokenIsElevated, Name, CommandLine, Exe,
            token(pid=int(int=Pid)) as TokenInfo,
            hash(path=Exe) as Hash,
            authenticode(filename=Exe) AS Authenticode,
            Username, Memory.WorkingSetSize AS WorkingSetSize
        FROM ProcList
        WHERE Name =~ ProcessRegex
            AND Pid =~ PidRegex
            AND Exe =~ ExePathRegex
            AND CommandLine =~ CommandLineRegex
            AND Username =~ UsernameRegex
            AND NOT if(condition= UntrustedAuthenticode,
                        then= Authenticode.Trusted = 'trusted' OR NOT Exe,
                        else= False )

</code></pre>

