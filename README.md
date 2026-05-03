# 30-day-system-engineering-mission
This is a 30-day engineering mission with outputs, constraints, and failure drills. A day-by-day 30-day execution plan with exact commands, configs, and traps. If we follow this properly, we won’t just “learn Linux/Docker/networking”—we’ll operate systems.

## OPERATING RULES (READ FIRST)

-   Every day = **build → break → fix → document**
-   No passive learning beyond 20% of the time
-   Everything goes into **Git**
-   If it’s not documented, it doesn’t exist


*If you’re not a software engineer. That’s not a weakness. It’s a design constraint.
So we adapt the plan to **infra-first execution with minimal, controlled application complexity**.
You don’t need to “build apps.”
You need to **run, connect, and debug services**.*


## HOW WE FIX THE ORIGINAL PLAN

The original version assumed you’d handle apps easily. That’s unrealistic.

So we simplify:

-   Use **pre-built apps** (no coding)
-   Focus on **deployment, connectivity, failure, recovery**
-   Treat apps as **black boxes**


## YOUR CORE STACK (SIMPLIFIED)

Instead of Node/Python coding, you’ll use:

-   Web: Nginx (static + reverse proxy)
-   App: Simple HTTP container (prebuilt)
-   DB: MySQL
-   Optional: Admin UI (phpMyAdmin)
