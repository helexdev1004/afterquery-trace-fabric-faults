# Trace flagged fabric faults through to finished panels

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/trace-fabric-faults` — **Operations / Supply chain**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

A customer has returned upholstery covers with weaving faults in them, and the cutting room has to say which cut panels were affected before the rest of the order ships. The mill flagged every fault when it inspected the rolls; the cutting room has the cut plan for every roll it laid up. Nobody has yet put the two together.

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/recall.json` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 6 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 300s |
| **Tags** | `textiles`, `quality-control`, `record-forensics`, `geometry`, `python` |

## Why it is hard

Tracing a mill's flagged faults through to the panels they were cut into is real and ordinary work. Quality technicians and cutting-room supervisors in upholstery, automotive trim and technical-textile plants do it whenever a customer complaint or a mill's late fault notice has to be turned into a recall of cut parts, and they do it from exactly these leavings: the mill's inspection reports, the cut plans, and the room's written procedure. The awkward parts of it here - an inspector who books in yards, another in inches, a third who writes across before along, a fourth whose counter runs from the last seam - are the ones a mill's reports actually have in them rather than invented ones.

<details>
<summary>Full note</summary>

The data is synthetic and was built truth first. 15 rolls were woven with their faults and their mill seams before anything was recorded, laid up and cut to a plan, and each of 645 faults landed in a known panel or in the waste. The inspection reports and the cut plans were written afterwards, the way five inspectors and the cutting room would have kept them. Every answer is therefore exact by construction rather than a judgement made later, and the records are static files, so nothing is computed at marking time. Forty faults are each to be traced to a panel, marked exactly, and no more than 4 may be wrong. Naming one panel for all forty gets 1. The arithmetic is not withheld. PROCEDURE.md gives all of it: how a fault is booked along and across a roll, how a roll is laid up and trimmed, that the lay is parted at a seam, how a cut plan gives its panels and numbers them, which panel takes a fault on a cut line, and the length of a yard and an inch. The work is in the doing. Forty faults have to be carried from the inspection frame onto the lay and found among 2081 panels, and the answer is a panel rather than a quantity, so there is nothing to fit and nothing to regress. Taking the records exactly as they stand traces 4 of 40. Five things about the records have to be got from the records themselves. None is written down anywhere, and each is settled by something in the records that cannot be right otherwise. The first is that one inspector books the distance along the roll in yards, so that some of his faults lie beyond the end of the roll. The second is that one inspector books the distance across the roll in inches, so that his faults stand forty times the roll's width from the selvedge. The third is that one inspector writes across before along. The fourth is that one inspector runs his counter from the last seam he passed and not from the head of the roll, so that his distances fall back at every seam. The fifth is not any inspector's habit, and it is the one that decides the task. A roll comes off the inspection frame wound up with the end the inspector saw first at its core. Laid up straight from the store it therefore comes off the tube tail first: turned end for end, and - since cloth is always handled face up - turned side for side as well, so that a fault booked at a distance along and across the roll lies at the roll's length less the one and its width less the other on the lay. A roll that was re-rolled between the mill and the table comes off head first again. Nothing in the records says which rolls were: 10 of the 15 were laid up tail first, two of every inspector's three, so it is nobody's habit and no inspector's rolls can be checked against another's. PROCEDURE.md says how the mill books a fault and how a cut plan is drawn, and says nothing of which end of a roll reaches the table first; what settles it is the mill seams. PROCEDURE.md records that a seam is never cut into a panel and that the lay is parted at it. Of the 45 seams the inspectors booked, 45 stand in a parting of the lay with their roll taken the right way about, and 45 stand inside a panel with it taken the wrong way. Taking every roll to have been laid up the way it was inspected costs 23 of the 40 faults - and nothing goes visibly wrong for a solver who does. Every one of the forty lands in a panel whichever way its roll is taken, a plausible wrong panel rather than the waste, and of all the mill's faults 27 fall in the waste with every roll taken the right way about and 30 with every roll taken as inspected, so the waste tells nothing either. Unlike the four inspectors' habits, which can each be checked against the rolls of the inspector who kept to the procedure, there is no clean set of rolls for this one. tests/gates.py scores all 64 ways of reading those five. Every figure quoted here comes from its output. The whole account traces all 40. The best account that is not the whole one traces 31, so missing any single one of them fails. 4 wrong are allowed, which leaves room for a slip and still sits 5 clear of every partial account. It also refuses to pass unless the seams really do settle how every roll was laid up, since a reading the records did not settle would be a guess and not a finding; and unless no fault to be traced stands within two centimetres of a cut line, so that neither the rule for a fault on a cut line nor a yard's or an inch's rounding decides an answer.

</details>

## How it is verified

The deliverable is a data file. The verifier reads /app/recall.json and never executes, imports or loads anything the submission wrote beyond parsing it as JSON, so there is nothing to sandbox and no way for a submission to influence its own score. 40 answers are marked, one for each fault in recall.txt, each a panel number matched exactly as text.

<details>
<summary>Full note</summary>

No more than 4 may be wrong. The sealed panels live only in the verifier image at /tests/truth.json; the agent's container carries the procedure, the inspection reports, the cut plans, the roll list and the list of faults, and nothing else. /tests is mode 700 in the verifier image, set at build time so it holds under a read-only root, and the agent's container has no mount of it at all; the verifier's log directory is likewise unreachable from the agent's side. There is nothing in the agent's container against which a solver could check an answer. No fault's panel is attested anywhere, no total has to come out and nothing has to balance, so a wrong set of panels is exactly as self-consistent as the right one. The panels have to be worked out; they cannot be searched for. tests/gates.py is shipped and can be re-run. It scores all 64 readings of the records against the sealed panels and refuses to pass if the whole account does not trace every fault, if any other account comes within the allowance, or if guessing does. It reports the whole account at 40, the best account that is not it at 31, the records taken as they stand at 4, and one panel named for everything at 1. The records are static files from a seeded generator, so nothing is computed at marking time and the answer cannot drift between runs. The reference was run against the shipped bundle in the task's own container and traced 40 of 40; an empty container scores 0.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/trace.py reads only what the agent's container carries and writes /app/recall.json. With solution/cutroom.py and solution/ref.py it is under three hundred lines and imports nothing outside the standard library. It reads the inspection reports, the cut plans and the roll list, and finds the inspectors' habits from the records rather than assuming them: the inspector whose distances along stop at a roll's width while his distances across run its length, the one whose faults stand further across than the roll is wide, the one whose faults lie beyond the end of the roll, and the one whose distances fall back after every seam. It puts each right, working in exact fractions and rounding to the centimetre only at the end. For every roll it then carries the mill's seams onto the lay both ways about - as inspected, and turned end for end and side for side - and keeps the way that leaves no seam inside a panel. It carries each fault onto the lay the same way and names the panel whose rectangle holds it, a fault on a cut line going to the panel beyond the line. Every quantity is a whole number of centimetres once converted and every comparison against a cut line is made exactly, so nothing turns on floating point and two correct implementations must agree to the panel. Run against the shipped records it traces 40 of 40.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
