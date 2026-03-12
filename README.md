# Sweet Crust Bakery - IBM Watson Assistant Chatbot

## Team Members
- Sai Nandini Tata
- Labina Tasfia
- Mrunal Bhosale

## Live Demo
**[Launch Chatbot](https://sites.google.com/colorado.edu/stat-5350midterm/home)**

## Scenario
Sweet Crust Bakery Assistant is a customer-service chatbot for a micro-bakery. It helps users place pickup orders, browse the menu, check hours and specials, ask about allergens, and get contact information. The bot supports slot-filling for orders and handles digressions mid-flow with smooth return.

## Features

### Intents (17)
Each intent has 5+ training examples.

| Intent | Purpose | Required Category |
|--------|---------|-------------------|
| greeting | Welcome and small talk | Greeting / small talk |
| goodbye | End conversation politely | Greeting / small talk |
| help | Describe bot capabilities | "What can you do?" |
| order | Place a bakery order | Primary task |
| cancel_order | Cancel an existing order | Follow-up (modify/cancel) |
| human_agent | Escalate to a real person | Escalation |
| Fallback (anything_else) | Handle unrecognized input | Fallback handling |
| menu | View available items | Supporting |
| hours | Check business hours | Supporting |
| prices | Get pricing info | Supporting |
| specials | View current deals | Supporting |
| allergens | Allergy and dietary info | Supporting |
| pickup | Pickup availability and process | Supporting |
| location | Bakery address and directions | Supporting |
| contact | Phone/email info | Supporting |
| confirm_yes / confirm_no | Order confirmation handling | Supporting |
| thank_you | Gratitude response | Supporting |

### Entities (10)

**System Entities (2):**
| Entity | Purpose | Example Inputs |
|--------|---------|----------------|
| @sys-date | Capture pickup date | "tomorrow", "Friday", "March 12th" |
| @sys-time | Capture pickup time | "3 pm", "noon", "10:30 AM" |

**Custom Entities (8):**
| Entity | Purpose | Example Values |
|--------|---------|----------------|
| @bakery_item | Item to order | cake, muffin, croissant, macaron, brownie, cookie |
| @quantity | How many items | one, two, half dozen, dozen, two dozen |
| @size | Item size | small, medium, large, extra large |
| @flavor | Flavor preference | chocolate, vanilla, strawberry, red velvet, caramel, lemon |
| @dietary | Dietary/allergy needs | gluten free, vegan, nut free, dairy free, sugar free |
| @occasion | Event type for custom orders | birthday, wedding, graduation, anniversary |
| @delivery_area | Delivery location | downtown, suburbs, nearby |
| @order_id | Order reference number | ORD-1234, SCB-1234, #1234 |

### Slot-Filling Flow (4 Slots)

The order flow is triggered by the `#order` intent and collects:

| Slot | Variable | Entity | Prompt |
|------|----------|--------|--------|
| 1. Item | `$order_item` | @bakery_item | "What bakery item would you like?" |
| 2. Quantity | `$order_quantity` | @quantity | "How many would you like?" |
| 3. Pickup Date | `$pickup_date` | @sys-date | "What date would you like to pick up your order?" |
| 4. Pickup Time | `$pickup_time` | @sys-time | "What time would you like to pick up your order?" |

- Slots re-prompt if input is missing or invalid
- Digressions are allowed mid-flow (`digress_out_slots: allow_all`)
- **Confirmation step**: *"Here's what I captured: Item: $order_item, Quantity: $order_quantity, Pickup Date: $pickup_date, Pickup Time: $pickup_time. Is that correct?"*
- User can confirm (`#confirm_yes`) or restart (`#confirm_no`)

### Context Variables (6)
| Variable | Purpose |
|----------|---------|
| `$current_flow` | Tracks active flow ("ordering" or "none") to control digression behavior |
| `$order_item` | Stores the selected bakery item |
| `$order_quantity` | Stores quantity requested |
| `$pickup_date` | Stores pickup date |
| `$pickup_time` | Stores pickup time |
| `$order_summary` | Stores full order summary referenced in confirmation |
| `$awaiting_confirmation` | Controls whether bot is waiting for yes/no confirmation |

Context is referenced later in responses (e.g., confirmation message uses `$order_item`, `$order_quantity`, etc.) and in conditions (e.g., `$current_flow == "ordering"` to trigger digression nodes).

### Digressions (3)
During the ordering flow, users can ask side questions without losing progress:

| Digression | Trigger | Behavior |
|------------|---------|----------|
| Hours | `#hours && $current_flow == "ordering"` | Answers hours, then says "Now, let's continue with your order." |
| Specials | `#specials && $current_flow == "ordering"` | Answers specials, then says "Now, back to your order - please continue." |
| Allergens | `#allergens && $current_flow == "ordering"` | Answers allergen info, then says "Now, back to your order - please continue." |

All digression nodes have `digress_in: returns` and `digress_out: allow_all` enabled.

## Deployment
Deployed via IBM Watson Assistant Web Chat embedded on a Google Sites page.

## Documentation
- [`design document.docx`](design%20document.docx) - Full design document with scenario overview, feature details, and conversation transcripts

## Files
| File | Description |
|------|-------------|
| `Bakery-dialog (1).json` | Watson Assistant dialog skill export (intents, entities, dialog nodes) |
| `Bakery_Intents.csv` | Intent training data (17 intents, 5+ examples each) |
| `Bakery_Entities.csv` | Custom entity definitions with synonyms |
| `design document.docx` | Design document with conversation transcripts |
