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

### Intents (18)
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
| flavors | Ask about available flavors | Supporting |
| pickup | Pickup availability and process | Supporting |
| location | Bakery address and directions | Supporting |
| contact | Phone/email info | Supporting |
| confirm_yes / confirm_no | Order confirmation handling | Supporting |
| thank_you | Gratitude response | Supporting |

### Entities (11)

**System Entities (2):**
| Entity | Purpose | Example Inputs |
|--------|---------|----------------|
| @sys-date | Capture pickup date | "tomorrow", "Friday", "March 12th" |
| @sys-time | Capture pickup time | "3 pm", "noon", "10:30 AM" |

**Custom Entities (9):**
| Entity | Purpose | Example Values |
|--------|---------|----------------|
| @bakery_item | Item to order | cake, muffin, croissant, macaron, brownie, cookie |
| @quantity | How many items | one, two, three, four, five, half dozen, dozen, two dozen |
| @size | Item size | small, medium, large, extra large |
| @flavor | Flavor preference | chocolate, vanilla, strawberry, red velvet, caramel, lemon |
| @dietary | Dietary/allergy needs | gluten free, vegan, nut free, dairy free, sugar free |
| @time | Pickup time slots | 8 am, 9 am, 10 am, 11 am, 12 pm, 1 pm, 2 pm, 3 pm |
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

## Analytics: Likely Failure Points and Improvement Plan

### Intent Overlap and Confusion Risks

1. **`#cancel_order` vs `#confirm_no`**: The word "Cancel" is a training example for `#confirm_no`, but "Cancel my order" trains `#cancel_order`. If a user says just "cancel" during the order confirmation step, Watson may route to `#confirm_no` (restart the order) rather than `#cancel_order` (abandon it entirely). This ambiguity is the highest-risk confusion point in the bot.

2. **`#contact` vs `#human_agent`**: "How do I get in touch" (`#contact`) and "I need to speak with someone" (`#human_agent`) share similar phrasing. A user saying "I want to talk to someone about my order" could match either intent. Adding more distinctive training examples (e.g., emphasizing "phone/email" for contact vs "transfer/escalate" for human agent) would reduce overlap.

3. **`#menu` vs `#order`**: "What do you have" (`#menu`) and "I want to buy" (`#order`) can blur when a user says something like "I want a chocolate cake" - this could be browsing or ordering. Currently the bot handles this reasonably since the order flow uses entity detection (`@bakery_item`), but edge cases exist.

4. **`#prices` vs `#menu`**: "How much for a brownie" could match `#prices` or `#menu` depending on confidence scores. Both intents deal with product information but from different angles.

5. **`#hours` vs `#pickup`**: "When can I pick up my order" could match `#hours` (about operating times) or `#pickup` (about the pickup process). The training examples are distinct enough, but real user phrasing often blends these concepts.

### Slot Validation Gaps

- **Unrecognized items**: If a user requests an item not in `@bakery_item` (e.g., "pizza", "bread"), the slot re-prompts but does not explicitly tell the user what items are available. Adding a "not found" handler that lists valid options would reduce user frustration.
- **Quantity edge cases**: The `@quantity` entity uses a custom list (one, two, dozen, etc.) rather than `@sys-number`. Inputs like "5", "seven", or "15" are not recognized, causing unnecessary re-prompting. Supplementing with `@sys-number` as a fallback would improve coverage.

### Unused Entities

Several entities are defined but not referenced in any dialog node or slot: `@flavor`, `@size`, `@occasion`, `@delivery_area`, and `@order_id`. These were built for future features (custom cake orders, delivery, order status lookup) but are currently dormant. Integrating them into the ordering flow or a dedicated custom-cake flow would add depth.

### Fallback Behavior

The fallback node uses a `sequential` selection policy with 3 messages. After the third unrecognized input, the bot repeats the last generic message ("I didn't get your meaning") with no further guidance. Improvement: after 2-3 consecutive fallbacks, the bot should proactively offer the help menu or escalate to a human agent using `$escalation_requested`.

### Improvement Plan

| Priority | Improvement | Impact |
|----------|------------|--------|
| High | Add counterexamples to disambiguate `#cancel_order` vs `#confirm_no` | Reduces the most likely misrouting |
| High | Add "not found" slot handlers listing valid options when entity match fails | Improves UX during ordering |
| Medium | Supplement `@quantity` with `@sys-number` fallback | Accepts numeric inputs like "5" or "seven" |
| Medium | Add more training examples to `#contact` and `#human_agent` to sharpen boundaries | Reduces overlap |
| Medium | Implement escalation after repeated fallbacks using context tracking | Prevents user frustration loops |
| Low | Build custom cake flow using `@flavor`, `@size`, `@occasion` entities | Leverages unused entities, adds feature depth |
| Low | Add order status lookup using `@order_id` entity | Enables "check my order" extra credit flow |

## Documentation
- [`design document.docx`](design%20document.docx) - Full design document with scenario overview, feature details, and conversation transcripts

## Files
| File | Description |
|------|-------------|
| `Bakery.json` | Watson Assistant dialog skill export (intents, entities, dialog nodes) |
| `Final_intents.csv` | Intent training data (18 intents, 5+ examples each) |
| `Final_entities.csv` | Custom entity definitions with synonyms |
| `design document.docx` | Design document with conversation transcripts |
