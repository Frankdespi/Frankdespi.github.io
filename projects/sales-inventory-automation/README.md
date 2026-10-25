# Sales & Inventory Automation

A small juice business was tracking sales and stock by hand. I built an n8n automation so the owner only logs sales, batches, and costs in Google Sheets. The spreadsheet does the arithmetic, and n8n does the watching.

![The n8n canvas for the Sales & Inventory Automation, showing six lanes. Lane 1 is a low-stock alert, lane 2 an 8 pm summary, lane 3 a 6 pm no-sales reminder, lane 4 a Sunday weekly roll-up, lane 5 an error alert, and lane 6 a weekly backup.](canvas.png)

*The n8n canvas: six lanes, each with its own trigger. The screenshot shows workflow structure only. No credentials, names, or figures appear in it.*

## What the owner does

The owner logs sales, batches, and costs in Google Sheets, picking the flavour and size from dropdowns. Nothing else is required.

## What happens automatically

| Lane | Trigger | Steps | Result |
| --- | --- | --- | --- |
| 1. Low-stock alert | A new sale row is added (Google Sheets trigger) | Get Stock Row, Is Low (IF), Telegram Alert | A Telegram message when a flavour, or its stickers, run low |
| 2. End-of-day summary | Schedule, 8 pm | Read Daily_Log, Read stock, Build Summary (Code), Telegram Summary, Gmail Summary | Bottles sold, revenue, top seller, and exactly which flavours and sizes were sold, on Telegram and by email |
| 3. No-sales reminder | Schedule, 6 pm | Read Daily_Log, Count Today (Code), Nothing Logged (IF), Telegram Reminder | A nudge if nothing has been logged yet |
| 4. Weekly roll-up | Schedule, Sunday 9 pm | Read Daily_Log, Read Costs, Weekly Totals (Code), Save Weekly Summary (append to the sheet), Telegram Weekly | Revenue, costs, profit, and flavours sold, saved to the sheet and sent to the owner |
| 5. Error alert | Error Trigger | Telegram Error | A message if anything breaks |
| 6. Weekly backup | Schedule, Sunday 11 pm | Copy Sheet Backup (Google Drive) | A copy of the sheet saved in Google Drive |

Every lane ends with a Track Time Saved node.

## How the lanes connect

```mermaid
flowchart TB
    subgraph L1["Lane 1: low-stock alert"]
        direction LR
        a1["New Sale Row<br/>Google Sheets trigger"] --> a2["Get Stock Row"] --> a3{"Is Low?"}
        a3 -- true --> a4["Telegram Alert"] --> a5["Track Time Saved"]
    end
    subgraph L2["Lane 2: end-of-day summary, 8 pm"]
        direction LR
        b1["Schedule 8pm"] --> b2["Read Daily_Log"] --> b3["Read stock"] --> b4["Build Summary<br/>Code"] --> b5["Telegram Summary"] --> b6["Gmail Summary"] --> b7["Track Time Saved"]
    end
    subgraph L3["Lane 3: no-sales reminder, 6 pm"]
        direction LR
        c1["Schedule 6pm"] --> c2["Read Daily_Log"] --> c3["Count Today<br/>Code"] --> c4{"Nothing Logged?"}
        c4 -- true --> c5["Telegram Reminder"] --> c6["Track Time Saved"]
    end
    subgraph L4["Lane 4: weekly roll-up, Sunday 9 pm"]
        direction LR
        d1["Schedule Sunday 9pm"] --> d2["Read Daily_Log"] --> d3["Read Costs"] --> d4["Weekly Totals<br/>Code"] --> d5["Save Weekly Summary<br/>append to sheet"] --> d6["Telegram Weekly"] --> d7["Track Time Saved"]
    end
    subgraph L5["Lane 5: error alert"]
        direction LR
        e1["Error Trigger"] --> e2["Telegram Error"]
    end
    subgraph L6["Lane 6: weekly backup, Sunday 11 pm"]
        direction LR
        f1["Schedule Sunday 11pm"] --> f2["Copy Sheet Backup<br/>Google Drive"] --> f3["Track Time Saved"]
    end
    L1 ~~~ L2
    L2 ~~~ L3
    L3 ~~~ L4
    L4 ~~~ L5
    L5 ~~~ L6
```

## Three things I would tell anyone building something similar

1. **Let the spreadsheet do the arithmetic and let automation handle the reporting.** Fewer moving parts means fewer things to break.
2. **Give every node a unique name when several workflows share one canvas.** Every node here carries its lane prefix, from L1 to L6, so no two nodes ever share a name.
3. **Test sheet protection from a second account.** Owners are never blocked by their own locks.

## Handover

I also wrote a short daily guide so the owner can run the system without me. A system nobody understands is a system nobody uses.

## Tools

n8n, Google Sheets, Telegram Bot API, Gmail, Google Drive, JavaScript Code nodes.

## About the data

The business name and its figures are kept private, and no credentials appear anywhere in this folder.
