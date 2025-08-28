# Building Actions for All Objects

-----

This activity is designed to give you hands-on experience with the Actions framework and encourage you to think about how you can turn a static data model into a dynamic, operational tool.

## Step-by-Step Guide for Orders

Here is a practical, step-by-step guide to creating a **Modify** action that updates an order. I've also included an extremely detailed scribe document for you to utilize here [Create and Configure Action Types and Python Functions in Ontology](https://scribehow.com/viewer/Create_and_Configure_Action_Types_and_Python_Functions_in_Ontology__559incNdT5ugRId7m7tzGA)

**Step 1: Enabling Edits on the Order Object Type**

  * **Navigate to Ontology Manager**: Go to the Ontology Manager and select the **Order** object type you created in the previous activity.
  * **Enable Edits**: Go to the **Datasources** tab and toggle on the option to **Enable edits**. Remember to click **Save** to apply the changes.

**Step 2: Creating a Modify Action**

  * **Create a New Action**: In Ontology Manager, go to the **Actions** tab and click **+ New Action Type**.
  * **Select Modify Action**: Name the action **Mark as Shipped**. The purpose of this action is to update the status of a specific order.
  * **Define Parameters**: You will need to configure a form that a user can fill out to update the order.
      * `orderNumber`: This is the **Primary Key** of the object and will be automatically populated from the selected **Order** object.
      * `status`: This property should be a static value, uneditable option: **"Shipped"**.
      * `shippedDate`: This property is current a type of String which makes it not as easy to update using something like a date calendar picker. For now, we'll just manually have to type in the date it shipped (ie. for today 2025-08-28).
  * **Regex**: We can levereage a regex expression, click on the parameter **Shipped Date** then within the constraints section under User Input check the "Matching regex" to make sure our format for shipped date at least matches what we would expect for YYYY-MM-DD by pasting this expression into the field
      * `^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$`
      * Failue Message: `Must be in the YYYY-MM-DD format`

**Step 3: Create a Function to Calculate Shipping Time**
In this section, you will create a Python function that calculates the time it takes for an order to ship. You will then be able to leverage this function to analyze shipping metrics for each order.

**Instructions:**

  * Paste the following code into the `my_function.py` file. This function will take an **Order** object as input and calculate the difference in days between the `orderDate` and `shippedDate`. If the `shippedDate` is not present, it will return **"Not Applicable"**.

<!-- end list -->

```python
from functions.api import function
from ontology_sdk.ontology.objects import Order
from datetime import datetime


@function()
def calculate_shipping_days(order: Order) -> str:
    """
    Calculates the number of days it took to ship an order.
    If the order has not yet been shipped, returns "Not Applicable".

    Args:
        order: The Order object to calculate shipping days for.

    Returns:
        A string indicating the number of days to ship or "Not Applicable".
    """
    if order.shipped_date and order.order_date:
        # Assuming the date strings are in 'YYYY-MM-DD' format.
        # Adjust the format code if your date string is different.
        try:
            shipped_date_obj = datetime.strptime(order.shipped_date, '%Y-%m-%d')
            order_date_obj = datetime.strptime(order.order_date, '%Y-%m-%d')
            days = (shipped_date_obj - order_date_obj).days
            return f"{days} days"
        except (ValueError, TypeError):
            # Handle cases where string format is incorrect or data is not as expected
            return "Invalid date format"
    else:
        return "Not Applicable"

```

  * Save the file and commit your changes.
  * Don't forget to `Resource Import` on the left panel & add or `Order` object type and install the SDK.
  * Check the `Functions` tab on the bottom of the browser and see if you're function is viewable without any errors. We should be able to preview to test prior to tagging & verisioning.

**Step 4: Using the Action**

  * **Navigate to Object View**: Open the Object View and find an Object with a status "In Progress". We can add the widget to hand our **Mark as Shipped** action within the Object View.
  * **Execute the Action**: Select the **Order** object, and from the object's details panel, you should see your **Mark as Shipped** action.
  * **Run and Observe**: Run the action. The `shippedDate` will be automatically populated, and the status will be updated to "Shipped." This live update demonstrates how Actions close the loop between data and operations.

## Open-Ended Challenge for Other Objects

Now you will apply the concepts from the previous steps to build basic operational layers for the other objects in your Ontology. This activity is designed to give you hands-on experience with the Actions framework and encourage you to think about how you can turn a static data model into a dynamic, operational tool. 

Don't forget to attempt some python functions as well, feel free to converse with the **AIP Assist** to help with some ideas for new python functions for these other objects.

**For the Car (Product) and Order Details Objects:**

  * **Task**: Explore these object types in Ontology Manager and Object Explorer.
  * **Goal**: Think about what actions a user might need to perform on these objects. For example, a user might want to update a car's price, or a manager might need to modify the quantity of a line item in an order. Use your knowledge of **Modify** and **Create** actions to build one new action for each of these object types.

**For the Customer and Order Objects:**

  * **Task**: Create actions to handle the lifecycle of a new customer and a new order.
  * **Goal**:
      * **Create a Create Action for a new Customer**: Build a form that allows a user to add a new customer, including their contact information.
      * **Create a Create Action for a new Order**: Build a form that allows a user to create a new order and link it to an existing **Customer** object.

# Advanced Challenge

-----

This section provides an opportunity to tie together all the concepts from the previous activities, moving from simple object and link creation to a complex, multi-step pipeline that builds on external data.

**Task**: Your objective is to enrich your Ontology by connecting the **Car** object type to public financial data.

**Prerequisites**: This challenge assumes you have access to the SEC EDGAR log files from your S3 bucket and a separate CSV file that maps car manufacturers to their Central Index Keys (CIK).

**Hint**: You will need to use **Pipeline Builder** to create a new dataset that acts as a bridge. A key step is to transform the raw `uri_path` from the log files to extract the CIK. You will also need to create a new **Company** object type backed by your mapping CSV. Your final pipeline should connect the **Car** and **Company** objects via a new **Manufactures** link type, joining on the car's make (which you would extract from the `productName` field) and the company's CIK.

This demonstrates how the Ontology can be progressively enriched, transforming a simple **Company** object into a hub for related financial information and showcasing the power of Foundry's "closed-loop" system.

