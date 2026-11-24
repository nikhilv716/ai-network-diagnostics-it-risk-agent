# AI-Powered Network Diagnostics & IT Risk Assessment Agent

*(Subject to changes)*

A University of Houston Computer Information Systems project by Nikhil Varghese.

I built a local lab to investigate a question an IT team might face: if an order-entry service appears to be down, what can we verify before reporting an outage? The project uses Codex and a custom diagnostic skill to check DNS, test the connection to a web service, and interpret what the application reports about its own health.

In two of the three experiments, the server returned `HTTP 200 OK`. The healthy service reported `"status":"ok"`, while the degraded service reported `"status":"degraded"`. Reading the response body changed the diagnosis even though the HTTP status stayed the same.

[Read the Milestone 2 report](Nikhil_Varghese_Milestone_2/Milestone_2_Result_and_Analysis.pdf) · [View the diagnostic skill](Nikhil_Varghese_Milestone_2/.agents/skills/diagnose-order-entry/SKILL.md) · [Browse the evidence](Nikhil_Varghese_Milestone_2/evidence/)

## Test results

| Scenario | What the tools showed | What the evidence supports |
| --- | --- | --- |
| [1. Healthy baseline](Nikhil_Varghese_Milestone_2/evidence/Milestone_2_Healthy_Baseline.txt) | DNS returned `127.0.0.1`, port `8765` had a TCP listener, and `/health` returned HTTP `200` with `"status":"ok"`. | The endpoint was reachable and reported healthy operation. The server log also recorded the request. |
| [2. Stopped HTTP service](Nikhil_Varghese_Milestone_2/evidence/Milestone_2_Scenario_2_Stopped_HTTP_Service.txt) | DNS still returned the expected address. `curl` exited with code `7`, and `ss` showed no listener on port `8765`. | Name resolution still worked, but the web service could not accept a TCP connection. The `000` shown by `curl` means it received no HTTP response. |
| [3. Unseen degraded condition](Nikhil_Varghese_Milestone_2/evidence/Milestone_2_Scenario_3_Unseen.txt) | After the lab services started, DNS resolved, the TCP listener was present, and `/health` returned HTTP `200` with `"status":"degraded"`. | The web service answered the request and reported a problem. The response did not identify the cause or establish whether an order would succeed. |

The Scenario 3 log preserves the agent’s first conclusion. In a fresh session, its DNS query was refused and it found no HTTP listener. That accurately described the environment at the time. After the supplied lab services started in that session, the agent repeated the same checks and observed the degraded response. The diagnostic skill stayed the same for both checks.

## How the agent works

1. **Check the name.** A Node.js script queries the lab DNS server at `127.0.0.1:15353` for the IPv4 address of `orders.test`.
2. **Test the web service.** `curl` requests `/health` on port `8765` and records the headers, response body, HTTP status, and exit code. It uses an explicit hostname-to-address mapping so the HTTP check can run independently of DNS.
3. **Verify the diagnosis.** The agent uses `ss` to see whether a process is listening on the expected port. It compares that result with the DNS answer and the HTTP output.
4. **Explain the business impact.** The agent reports what happened to the endpoint and what remains uncertain about order entry. These experiments only requested `/health`. Verifying that customers can place orders would require submitting a test order and checking whether it was accepted.

## Run the lab

Use a Linux environment with **Python 3**, **Node.js**, **curl**, **ss** (from `iproute2`), and **Codex**. The lab scripts use the Python and Node.js standard libraries. Run the services and Codex in the same Linux environment so they share the same loopback address.

From the repository root, open three terminals and run this command in each one:

```sh
cd Nikhil_Varghese_Milestone_2
```

**Terminal 1 — start DNS**

```sh
python3 lab/fixture.py dns
```

**Terminal 2 — start the healthy HTTP service**

```sh
python3 lab/fixture.py http
```

Once both services print `READY`, leave those terminals running.

**Terminal 3 — run the diagnostic agent**

Start Codex in the Milestone 2 directory and enter this prompt:

> Read and follow `.agents/skills/diagnose-order-entry/SKILL.md` to check `orders.test`. Run the DNS, HTTP, and TCP listener checks. Report the tool output, your diagnosis, and what the evidence establishes about order submission.

To reproduce the three scenarios, use the same prompt each time:

1. **Healthy baseline:** Run the diagnostic with both services active.
2. **Stopped HTTP service:** Press `Ctrl+C` in Terminal 2. Leave DNS running in Terminal 1, then rerun the diagnostic.
3. **Degraded response:** In Terminal 2, run `python3 lab/fixture_scenario_3.py http`. Once it prints `READY`, rerun the diagnostic and compare the response body with the baseline.

When finished, press `Ctrl+C` in each service terminal to stop the lab.

## Repository contents

| Folder | What’s inside |
| --- | --- |
| [Milestone 1](Nikhil_Varghese_Milestone_1/) | The initial diagnostic skill, lab scripts, early test evidence, screenshots, and progress report. |
| [Milestone 2](Nikhil_Varghese_Milestone_2/) | The three documented scenarios, the degraded-service script, the diagnostic skill used for testing, and the results and analysis report. |
