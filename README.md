# Purchase Validator 

**Custom Tag Template for Google Tag Manager**

Validates last purchase event and pushes results to dataLayer (Custom Tag Template for Google Tag Manager)

---

## Usage 

### Trigger
Create a new tag using this template (install manually as long as it is not part of the gallery) and fire it on your success page, using a trigger that depends on the unique URL or any other event that occurs on that page only, without using the `purchase` event. When checkig the URL, use something like "page loaded" instead of "page view" or other early events to make sure, the purchase is already present in the dataLayer when the tag executes. 

#### Block Page Reload
Whether you block a purchase from being sent on a page reload or not (GA is quite good at deduplication; at least within the same session), you should block the validator tag in order to avoid *"no purchase event"* errors on reloaded OSPs that do not contain a purchase event anymore (which would be the desirable behaviour). 

This can be achieved by adding a regular *JavaScript Variable* (not *Custom JavaScript*) for `window.performance.navigation.type` and check if the value equals *1* in a blocking trigger for the purchase validator tag. 

<img width="489" height="444" alt="image" src="https://github.com/user-attachments/assets/9d9655b0-b54a-476f-9fe4-75f106b873a5" />

## Options
There are some options to define the scope for validation:

- **"Value required"**: Check if a purchase without a value should raise an error. A value of 0 will still be accepted. If a value is present, the tag checks for `currency` having a value, too. 

- **"Tax / shipping required"**: Check if a purchase without a tax / shipping value should raise an error. A value of 0 will still be accepted. 

- **"Require item_id AND item_name "**: Items require either an item_id or item_name. Check this option if items with only one of them should raise an error. Will only be raised once for the first matching item, ignoring additional incomplete following items.

- **"Count Items"**: Adds the number of items to the result push

- **"Count Item Dimensions"**: If activated, the result will contain the total count of dimensions for all items + min and max count in a single item-

- **"Activate Warnings"**: If warnings are active, the result will contain info about missing recommended fields like price or quanitity and inform about wrong value types (string instead of integer / number).

- **"Log Result To Console"**: explains itself. 

## Size check
You can optionally define a size limit for the ecommerce object. Even if this does not reflect actual GA4 request payload size, you can use this option if you suspect problems caused by large payloads (e. g. from custom product dimension values).

## Using Results
When the tag fires, it looks for the most recent `purchase` event in the dataLayer and inspects it. If no event can be found, the result will include an error. 

If there is a `purchase`, the `ecommerce` object will be checked for several things: 

- a transaction_id must be present and have a value other than empty string, null or undefined (or "null" / "undefined" as a string)
- a value must be present, if the option is checked
- if a value is found, it has to be either a number or a string that can be parsed to a number
- if a tax and / or shippig value is required, an error occurs if not present or invalid. If a value is found, it has to be either a number or a string that can be parsed to a number and will be checked in any case (even if option is not checked)
- if a value is found, a currency must be present and set with a value
- items array must be present and not empty
- items must have at least an item_id or item_name
- if there is a quantity, it must be a valid number or string, that can be parsed to a number
- if there is a price, it must be a valid number or string, that can be parsed to a number
- if a size limit is defined and the stringified ecommerce object exceeds this size, an error will be added to the result    
- if there is no price or quantity or the format is not numeric, a warning will occur (if warnings are activated). 

## Output Format
After the tag is done, the results get pushed to the dataLayer, along with a `purchase_validation` event. The push includes a `check_results` object. This object has a readable `result_type` that you can use to compose an event name. 

The `transaction_id` will be added for reference (if there is one, otherwise *<NONE* will be the value here).

Number of errors and (if activated) warnings are also part of this object. If you want to transport the details as custom parameters for your result event (if you send one to Analytics or any other direction), there are `errorText` and `warningText` keys in the object, containing readable error messages (comma separated). 

### Possible error messages
- no purchase event
- no ecommerce object
- no transaction_id
- no value / tax / shipping
- no currency
- value / tax / shipping incorrect format
- no items
- empty items
- no item_id or item_name
- no item_id / no item_name (if both keys are required for every item)
- quantity incorrect string (ref)
- price incorrect string (ref)

### Possible Warnings
- value / tax / shipping not numeric
- no quantity (ref)
- quantity not numeric (ref)
- no price (ref)
- price not numeric (ref)

**Note**: item related warnings and errors occur only once, even if detected several times. Otherwise the text attributes would get too big to transport most of the times. The "ref" in brackets will include the item_id or item_name as reference.     

**Examples**:

```
//multiple problems
check_results: {
  result_type: "error",
  transaction_id: "<NONE>",
  errors: 2,
  errorText: "no transaction_id, quantity incorrect string (SKU_12345)",
  warnings: 3,
  warningText: "value not numeric, quantity not numeric (SKU_12345), no price (SKU_12345)"
}

//problems with items
check_results: {
  result_type: "error",
  transaction_id: "T_ERR0101",
  errors: 1,
  errorText: "quantity incorrect string (SKU_12345)",
  warnings: 2,
  warningText: "quantity not numeric (SKU_12345), no price (SKU_12345)"
}

//warnings, but no error
check_results: {
  result_type: "warning",
  transaction_id: "T_566789",
  errors: 0,
  errorText: "",
  warnings: 1,
  warningText: "quantity not numeric (SKU_XY)"
}

//all good:
check_results: {
  result_type: "success",
  transaction_id: "T_OKAY1",
  errors: 0,
  errorText: "",
  warnings: 0,
  warningText: ""
}

//no purchase found:
check_results: {
  result_type: "error",
  transaction_id: "<NONE>",
  errors: 1,
  errorText: "no purchase event",
  warnings: 0,
  warningText: ""
}

//example result with item activated stats
check_results: {
  result_type: "error",
  transaction_id: "T_23456",
  errors: 2,
  errorText: "no tax, no item_name",
  warnings: 1,
  warningText: "shipping not numeric",
  item_count: 2,
  item_dims_min: 18,
  item_dims_max: 19,
  item_dims_sum: 37
}

```
