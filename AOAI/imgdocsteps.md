# Take a picture and convert to Documentation, Steps for Code

## Use Case

- Take a picture of code and convert to documentation
- Take a picture and ask mode to write a code snippet
- Any design UX team and convert to code
- Only for concepts and ideation for now.
- Figma UX design images to code/steps/documentation

## Pre-requisites

- Azure subscription
- Azure Open AI GPT 4 vision
- Azure power apps

## Steps - Code

- I downloaded a image from google - public source just to test
- It's a sample figma sample image
- Now wanted to use prompt engineering to see if it can create steps by step directions for usage

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/imgtocodedocs1.jpg 'RagChat')

- Prompt 

```
Based on the image provided for Home screen can you provide step by step documentation?
```

- Response

```
Based on the provided image, here is the step-by-step documentation for creating a creative request from the Home screen:

Step 1: Navigate to the Home screen and click on the "+ Create Request" button located in the left sidebar menu, under the "Quick Access" section. This will open the "Create Request" modal.
v
Step 2: In the "Create Request" modal, click on the "Add CREA 18" button to begin adding items to your request. 

Step 3: A new modal will pop up titled "ADD CREA 18 ITEMS." Click on the "Add Items" button to proceed with adding individual items.

Step 4: After clicking "Add Items," another modal will appear titled "ITEM DETAILS." Here you will select the item type from a dropdown menu.

Step 5: Fill in the required fields for the selected item type. This includes details such as Size, Color, Material, Quantity, and other specifications.

Step 6: Enter any additional information in the "Additional Details" text box if necessary.

Step 7: If you have any files to attach, click on the "Upload" button to attach files relevant to the request.

Step 8: Review all the entered information for accuracy. 

Step 9: Once all item details are filled out correctly, click on the "Save" button to save the item to the request.

Step 10: Repeat Steps 4 through 9 for any additional items that need to be added to the request.

Step 11: After adding all the necessary items, you can review the list of items in the "ITEM DETAILS" modal.

Step 12: If you need to edit any item, click on the "Edit" button next to the respective item.

Step 13: To finalize the creative request, click on the "Submit" button. 

Step 14: A confirmation message will appear to confirm that your creative request has been successfully created.

Step 15: You can now close the modal by clicking on the "X" or clicking outside the modal area to return to the Home screen.

Please note that this is a hypothetical documentation based on the provided image and might not reflect the actual functionality of the software.
```

- Now prompt to write code
- Here is the prompt to convert to Type script code

```
Based on the image provided for Home screen can you write type script code?
```

- I am only going based on one screen at a time.
- Reposne

```
Certainly! Based on the Home screen image provided, it appears to be a multi-step form process for creating a request. Here's a TypeScript code structure that might represent the form and its steps. Please note that this is a high-level example and does not include all the detailed functionality that would be needed for a complete implementation.

```typescript
// Typescript code structure for a multi-step form process

// Define an interface for the form data
interface RequestFormData {
  itemsToAdd: number;
  itemDetails: ItemDetail[];
}

interface ItemDetail {
  itemId: string;
  quantity: number;
  color: string;
  size: string;
  additionalDetails: string;
}

// Main component that handles the steps and form data
class CreateRequestForm extends React.Component<{}, { currentStep: number, formData: RequestFormData }> {
  constructor(props: {}) {
    super(props);
    this.state = {
      currentStep: 1,
      formData: {
        itemsToAdd: 0,
        itemDetails: []
      }
    };
  }

  // Method to go to the next step
  nextStep = () => {
    this.setState(prevState => ({ currentStep: prevState.currentStep + 1 }));
  }

  // Method to go back to the previous step
  prevStep = () => {
    this.setState(prevState => ({ currentStep: prevState.currentStep - 1 }));
  }

  // Method to handle form input changes
  handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = event.target;
    // Update formData here based on input changes
  }

  // Method to add a new item detail
  addItemDetail = (itemDetail: ItemDetail) => {
    this.setState(prevState => ({
      formData: {
        ...prevState.formData,
        itemDetails: [...prevState.formData.itemDetails, itemDetail]
      }
    }));
  }

  // Method to submit the form
  submitForm = () => {
    // Submit formData to the backend or perform some action
  }

  render() {
    const { currentStep, formData } = this.state;

    return (
      <div>
        <h1>Create Request</h1>
        {/* Conditionally render steps based on currentStep */}
        {currentStep === 1 && <AddItemsStep formData={formData} handleChange={this.handleChange} nextStep={this.nextStep} />}
        {currentStep === 2 && <ItemDetailsStep formData={formData} addItemDetail={this.addItemDetail} nextStep={this.nextStep} prevStep={this.prevStep} />}
        {/* Additional steps can be added here */}
        {currentStep === 3 && <ReviewSubmitStep formData={formData} prevStep={this.prevStep} submitForm={this.submitForm} />}
      </div>
    );
  }
}

// Step components can be defined here with their specific form fields and logic
const AddItemsStep = ({ formData, handleChange, nextStep }) => {
  // Component logic and JSX for the first step
};

const ItemDetailsStep = ({ formData, addItemDetail, nextStep, prevStep }) => {
  // Component logic and JSX for the second step
};

const ReviewSubmitStep = ({ formData, prevStep, submitForm }) => {
  // Component logic and JSX for the review and submit step
};

// Render the main component
const rootElement = document.getElementById('root');
ReactDOM.render(<CreateRequestForm />, rootElement);
```

Please note that this code structure is only a starting point and does not include all the necessary components, styles, and error handling that would be required in a full application. You would need to further develop each step component (`AddItemsStep`, `ItemDetailsStep`, `ReviewSubmitStep`) with the appropriate form fields and logic based on the provided design.
```

- Now let's ask the same to write python code

```
Based on the image provided for Home screen can you write stream lit python web app code?
```

- Response

```
Certainly! Here is an example of how you might start to code a Streamlit web application with a layout similar to the one in the image provided. This code will create a basic structure with a sidebar and different sections to simulate the interface shown in the image.

```python
import streamlit as st

# Define the layout of the app
def main():
    st.sidebar.title("HOME")

    # Sidebar - Menu selection
    menu = ["Status", "My Requests", "New Creative Request", "Recent Orders"]
    choice = st.sidebar.selectbox("Menu", menu)

    # Main Content Area
    if choice == "Status":
        st.header("Status Overview")
        # Add content for Status Overview

    elif choice == "My Requests":
        st.header("My Requests")
        # Add content for My Requests

    elif choice == "New Creative Request":
        st.header("Create New Request")
        with st.form(key='new_request_form'):
            # Add form elements here
            item_name = st.text_input("Item Name")
            description = st.text_area("Description")
            quantity = st.number_input("Quantity", min_value=1, value=1)
            submit_button = st.form_submit_button(label='Submit')

            if submit_button:
                # Process the submitted form data
                st.write(f"Request submitted for {item_name} with quantity {quantity}.")

    elif choice == "Recent Orders":
        st.header("Recent Orders")
        # Add content for Recent Orders

    # Sidebar - Recent Orders List
    st.sidebar.header("Recent Orders")
    for i in range(1, 10):
        st.sidebar.text(f"Order {i}")

# Run the main function
if __name__ == "__main__":
    main()
```

To run this code, save it to a Python file (for example, `app.py`) and then run it using the following command in your terminal:

```bash
streamlit run app.py
```

Please note that this code is a basic starting point and does not include all the features shown in the image, such as modals and detailed form controls. You would need to extend the code with additional Streamlit widgets and logic to fully replicate the interface and functionalities shown.
```

- now we can change prompts for various different cases and see how it responds.