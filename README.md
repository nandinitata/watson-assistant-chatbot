# Sweet Crust Bakery - IBM Watson Assistant Chatbot

## Live Demo
**[Launch Chatbot](https://sites.google.com/colorado.edu/stat-5350midterm/home)**

## Scenario
A customer-service chatbot for a micro-bakery that helps users place orders, check hours, view specials, ask about allergens, and get contact information.

## Features

### Intents (17)
| Intent | Purpose |
|--------|---------|
| greeting | Welcome and small talk |
| help | Describe bot capabilities |
| order | Place a bakery order (primary task) |
| cancel_order | Cancel an existing order |
| human_agent | Escalate to a real person |
| menu | View available items |
| hours | Check business hours |
| prices | Get pricing info |
| specials | View current deals |
| allergens | Allergy and dietary info |
| pickup | Pickup availability and process |
| location | Bakery address |
| contact | Phone/email info |
| confirm_yes / confirm_no | Order confirmation handling |
| thank_you | Gratitude response |
| goodbye | End conversation |

### Entities (10)
- **System entities**: @sys-date, @sys-time
- **Custom entities**: @bakery_item, @quantity, @size, @flavor, @dietary, @occasion, @delivery_area, @order_id

### Slot-Filling Flow (4 slots)
The order flow collects:
1. **Item** - What bakery item (cake, muffin, croissant, etc.)
2. **Quantity** - How many
3. **Pickup Date** - When to pick up
4. **Pickup Time** - What time

Includes a confirmation step: *"Here's what I captured: Item, Quantity, Date, Time. Is that correct?"*

### Context Variables
- `$current_flow` - Tracks whether user is in the ordering flow
- `$order_item`, `$order_quantity`, `$pickup_date`, `$pickup_time` - Slot values
- `$order_summary` - Stores full order summary
- `$awaiting_confirmation` - Controls confirmation state

### Digressions
During the ordering flow, users can ask about:
- **Hours** - Bot answers and returns to the order
- **Specials** - Bot answers and returns to the order
- **Allergens** - Bot answers and returns to the order

## Deployment
Deployed via IBM Watson Assistant Web Chat on Google Sites.

## Files
- `Bakery-dialog (1).json` - Watson Assistant dialog export
- `Bakery_Intents.csv` - Intent training data
- `Bakery_Entities.csv` - Entity definitions
