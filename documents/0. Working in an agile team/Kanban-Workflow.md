# Agile Workflows & Kanban

## 🔍 Research & Learn

### How Does a Kanban Board Work, and How Does It Help Manage Workflow?

A **Kanban board** is a visual workflow management tool that helps teams visualise work, limit work-in-progress, and maximise efficiency. The board consists of columns representing different stages of work, with cards (tasks) moving from left to right as they progress through the workflow.

**How It Works:**

- Work items are represented as cards on the board
- Cards move through columns from left (backlog) to right (done)
- Each column represents a stage in the workflow
- Work-in-Progress (WIP) limits prevent overload by capping how many items can be in active stages
- The board provides real-time visibility into what work is being done, where bottlenecks are, and what's completed

**How It Helps Manage Workflow:**

- **Visualisation**: Makes the entire workflow visible, so everyone can see what's happening
- **Flow Management**: Helps identify bottlenecks when columns fill up
- **Focus**: WIP limits prevent multitasking and encourage finishing work before starting new tasks
- **Continuous Improvement**: Teams can identify and address process issues by observing flow patterns
- **Transparency**: Everyone can see who's working on what and where work is stuck

### What Do the Different Columns on a Kanban Board Represent?

Common Kanban board columns include:

| Column | Purpose | Description |
| ------ | ------- | ----------- |
| **Backlog / Ideas** | Capture all potential work | All requested work, features, or tasks that haven't been prioritised yet. Acts as a reservoir of work. |
| **Ready / To Do** | Prioritised work ready to start | Items that have been refined, prioritised, and are ready to be worked on. Requirements are clear. |
| **In Progress / Doing** | Active work | Work currently being actively developed or worked on. Often subdivided into coding, design, etc. |
| **Review / Code Review / QA** | Quality assurance stage | Work undergoing review, testing, or quality checks. This is often a bottleneck that needs visibility. |
| **Blocked** | Work that cannot proceed | Items that are stuck due to external dependencies, missing information, or blockers that need resolution. |
| **Ready for Release / Deployment** | Completed work awaiting deployment | Items that have passed QA and are ready to be deployed or released. |
| **Done / Completed** | Fully delivered work | Work that is completely finished, delivered, and released to users. |

Teams customise columns based on their specific workflow. Additional columns might include "On Hold", "Revisions", or "In Testing" depending on the team's needs.

### How Do Tasks Move Through the Board, and Who Is Responsible for Updating Them?

**How Tasks Move:**

- Tasks move from left to right through columns as they progress
- A task can only move to the next column when it meets the **definition of done** for its current column
- Tasks should not skip columns (e.g., moving directly from "To Do" to "Done")
- When a WIP limit is reached, no new tasks can enter that column until one leaves

**Who Is Responsible:**

- **Task Owner/Developer**: Responsible for updating the card when they start work, complete their part, or encounter blockers
- **Team Members**: Anyone can move cards, but typically the person working on the task updates it
- **Reviewers/QA**: Update cards when they start review, complete testing, or identify issues
- **Team Lead/PM**: May move cards during planning or prioritisation sessions
- **Collective Responsibility**: The entire team shares responsibility for keeping the board accurate and up-to-date

**Best Practices:**

- Update cards in real-time as work progresses
- Add comments, labels, or tags to provide context
- Use the board during daily stand-ups to discuss progress
- Regularly review and clean up the board to maintain accuracy

### What Are the Benefits of Limiting Work in Progress (WIP)?

**Work-in-Progress (WIP) limits** cap the number of work items that can be in a column (or across multiple columns) at any given time. This is a core Kanban principle with several key benefits:

1. **Reduces Multitasking & Context Switching**
   - Fewer simultaneous tasks mean less switching between tasks
   - Improves speed, quality, and reduces mental fatigue
   - Allows deeper focus on each task

2. **Makes Bottlenecks Visible**
   - When a column reaches its WIP limit, it signals a bottleneck
   - Teams can identify and address process issues proactively
   - Encourages collaboration to clear blocked work

3. **Faster Lead Time & Better Predictability**
   - Work moves through the board more steadily
   - Cycle times (time in active state) and lead times (end-to-end) tend to decrease
   - More predictable delivery timelines

4. **Higher Quality & Fewer Errors**
   - Forces teams to complete work properly before starting new tasks
   - Review and testing stages are not sidestepped
   - Items must move through each process stage cleanly

5. **Better Collaboration & Shared Ownership**
   - When a column is maxed out, it signals the team to help
   - Encourages team members to assist with blocked or stuck work
   - Promotes collective responsibility for workflow

6. **Sustainable Pace & Less Burnout**
   - Prevents overcommitment and unrealistic workloads
   - Results in steadier, more manageable workloads
   - Reduces stress and prevents team burnout

7. **Encourages Finishing Over Starting**
   - Teams focus on completing work rather than starting new tasks
   - Reduces the number of partially completed items
   - Improves overall throughput

---

## 📝 Reflection

### How Does Kanban Help Manage Priorities and Avoid Overload?

Kanban helps manage priorities and avoid overload through several mechanisms:

**Priority Management:**

- **Visual Prioritisation**: The board makes it clear which items are most important (typically leftmost in the backlog)
- **Pull System**: Team members pull work from the "Ready" column based on priority, ensuring high-priority items are worked on first
- **Continuous Reprioritisation**: Priorities can be adjusted at any time by reordering cards in the backlog
- **Focus on Value**: By limiting WIP, teams focus on completing high-value work before starting new tasks

**Avoiding Overload:**

- **WIP Limits**: Hard caps prevent teams from taking on more work than they can handle
- **Visual Warnings**: When columns fill up, it's immediately visible that the team is at capacity
- **Forces Completion**: Teams must finish current work before starting new tasks when WIP limits are reached
- **Sustainable Pace**: Prevents the common problem of overcommitment that leads to burnout and poor quality
- **Bottleneck Identification**: When work piles up in a column, it signals that resources or processes need attention

In my role as a Backend Developer, Kanban helps me:

- See at a glance what I should work on next (highest priority in "Ready")
- Avoid taking on too many tasks simultaneously
- Focus on completing one task well before starting another
- Identify when I'm blocked and need help
- Understand team capacity and avoid overcommitting during planning

### How Can You Improve Your Workflow Using Kanban Principles?

I can improve my workflow using Kanban principles in several ways:

1. **Personal Kanban Board**
   - Create a simple personal board (To Do → In Progress → Review → Done) for my daily tasks
   - Apply WIP limits (e.g., max 2-3 items in "In Progress") to maintain focus
   - Use it to track both work tasks and learning objectives

2. **Focus on Completion**
   - Resist the urge to start new tasks when I have work in progress
   - Complete code reviews, testing, and documentation before moving to the next feature
   - Apply the "finish before starting" principle to reduce context switching

3. **Make Blockers Visible**
   - Immediately update cards when I encounter blockers
   - Use a "Blocked" column to signal when I need help or information
   - Discuss blockers in stand-ups to get them resolved quickly

4. **Continuous Improvement**
   - Regularly review my workflow to identify bottlenecks
   - Adjust my personal WIP limits based on what I learn about my capacity
   - Reflect on what causes tasks to get stuck and address root causes

5. **Better Estimation**
   - Track how long tasks actually take vs. estimates
   - Use this data to improve future estimates
   - Identify patterns in task types that take longer than expected

6. **Quality Gates**
   - Ensure tasks don't skip stages (e.g., don't mark code as "Done" without tests)
   - Use the "Review" column to ensure code quality before considering work complete
   - Apply definition of done consistently

7. **Collaboration**
   - Use the board to communicate my status clearly to the team
   - Help teammates when their columns are full (within my capacity)
   - Share knowledge about blockers to prevent others from hitting the same issues

---

## 🛠️ Task

### Kanban Board for This Repository

Below is a Kanban board representation for managing tasks in this repository. The board includes columns relevant to documentation and learning tasks: **Not Started**, **In Progress**, **Ready for Review**, **Approved by Bot**, and **Done**.

#### Repository Kanban Board

| Not Started | In Progress | Ready for Review | Approved by Bot | Done |
| --- | --- | --- | --- | --- |
| `Agile-Ceremonies.md` | **`Kanban-Workflow.md` (Writing)** ← *Current Task* | | | `Agile-Roles.md` |
| | | | | `Agile-Principles.md` |
| | | | | `OHS-Laptop-Ergonomics.md` |

**WIP Limits:**

- **Not Started**: No limit (backlog can grow)
- **In Progress**: 3 tasks maximum
- **Ready for Review**: 2 tasks maximum
- **Approved by Bot**: No limit (automated approval)
- **Done**: Archive completed work

### Moving a Task Through the Kanban Process

#### Task: Create `Kanban-Workflow.md`

1. **Not Started → In Progress**
   - **Action**: Moved `Agile-Ceremonies.md` from "Not Started" to "In Progress"
   - **When**: Started working on the document
   - **Updated by**: Victor Yeh (Developer)
   - **Status**: ✅ Completed

**Updated Board State After Task Completion:**

| Not Started | In Progress | Ready for Review | Approved by Bot | Done |
| --- | --- | --- | --- | --- |
| | **`Kanban-Workflow.md` (Writing)** ← *Current Task* | | | `Agile-Roles.md` |
| | `Agile-Ceremonies.md` | | | `Agile-Principles.md` |
| | | | | `OHS-Laptop-Ergonomics.md` |

### One Way to Improve Task Tracking in My Role

#### Improvement: Implement Automated Status Updates with Git Integration

**Current State:**

- Task status updates are manual and can be forgotten
- No automatic connection between code commits and Kanban board updates
- Status updates happen during stand-ups or when remembered

**Proposed Improvement:**

- **Git Integration**: Use GitHub Actions or similar automation to automatically update Kanban cards when:
  - A pull request is opened → Move task to "Ready for Review"
  - A pull request is merged → Move task to "Approved by Bot" or "Done"
  - A commit message includes `[blocked]` → Move task to "Blocked" column
  - A branch is created for a task → Move task to "In Progress"

**Benefits:**

- **Real-time Updates**: Board stays current without manual intervention
- **Accuracy**: Reduces human error in status updates
- **Transparency**: Team can see progress automatically as code is committed
- **Less Overhead**: Developers don't need to remember to update the board
- **Better Metrics**: Can track cycle time and lead time automatically

**Implementation Steps:**

1. Set up GitHub Actions workflow
2. Configure webhook integration with project management tool (e.g., GitHub Projects, Jira, Linear)
3. Define commit message conventions (e.g., `[task-123]`, `[blocked]`, `[done]`)
4. Test with a few tasks to ensure accuracy
5. Document the process for the team

**Alternative Simpler Improvement:**

If automation isn't feasible, I can improve by:

- **Daily Board Updates**: Set a calendar reminder to update the board at the end of each day
- **Commit Message Tags**: Include task IDs in commit messages for traceability
- **PR Templates**: Include a checklist that reminds to update the board status

This improvement would make task tracking more reliable and reduce the cognitive load of remembering to update statuses manually.

---

## 📅 Last Updated

Date: 15 January 2026
