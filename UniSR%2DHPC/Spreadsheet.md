# Spreadsheet Guide
We thought it might be useful to keep track of who is using a cluster resource. This helps to understand how many resources are still available and allows users to better plan their resource usage according to their needs.

To do this, it was decided to maintain a shared spreadsheet where various computing resources can be "booked". **This booking is not official and is entirely informal, but each user is expected to honor their declarations to maximize collaboration.**

The spreadsheet can be accessed using this [link](https://docs.google.com/spreadsheets/d/1MnLAcGTaunwBftP2LH9d0P8F6RWABwIRjrp8GGBxhBQ/edit?usp=sharing). If you haven't been granted access to the document yet, feel free to request it: we will verify your identity and grant it to you.

# How to use the sheet

1. **Register**
	- Open the [sheet](#users) `Users` and fill it in with a unique `nickname`
	- Choose your color
2. **Book**
	- Open a [planner sheet](#resources) (e.g., `GPU Planner`)
	- Choose a free slot you prefer
	- Select or type your `nickname` inside it
	- The cell will be colored with the color you chose
	- Extend the cell to other boxes to enlarge your booking

# Resources
The spreadsheet is divided into three main sheets:
- **GPU planner**: dedicated to GPU booking on the cluster nodes. The code `ws01-gpu0` refers to `gpu0` of `workstation-01`, and so on.
- **CPU planner**: dedicated to booking CPU cores on the cluster. **For convenience, we decided to divide bookings into 8-core slots**. The column `ws01-s0` books an 8-core slot on `workstation-01`, for example.
- **RAM planner**: dedicated to booking RAM on the cluster. **For convenience, we decided to divide bookings into 8GB slots**. The column `ws01-s0` books an 8GB RAM slot on `workstation-01`, for example.

# Time slots
The planning sheets are divided into time slots: each row in the table refers to a specific time slot on a specific date. The sheets are designed to **update dynamically**, so the first row will always correspond to the current slot and the rows below to future slots.

Rows corresponding to past slots are automatically deleted from the planning sheets and logged in a hidden sheet.

> [!IMPORTANT]
> To prevent a user from booking all available slots for a specific resource, we decided to set a booking limit consistent with average training times: **a user can book a resource for a maximum of three days.**

# Users
Before booking a resource, it is necessary to enter your `First Name`, `Last Name`, and `Nickname` in the `Users` sheet. The `nickname` must be unique. If a booking contains a nickname not yet registered in the `Users` sheet, the booking will not be valid and the sheet will flag it as an error.

It is important to choose your personal color so that the slots related to your bookings are colored accordingly. This makes everything more readable and easier to interpret.

> [!WARNING]
> If you already have active bookings and decide to change your personal color, the color of those bookings will remain unchanged. Future bookings will have the new color as background.

# Protections
Most ranges are protected from edits, but unfortunately Google Sheets does not allow full control over changes. That is: any user can modify other users' bookings. Owner's trump card: edit logs, **forewarned is forearmed 😜.**