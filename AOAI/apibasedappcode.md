# Given a set of API"s create software web application code that uses the API"s.

## Introduction

- Create your microservice using the API"s provided.
- Now create new use cases based on the API's available.
- Create a new web application that uses the API's.
- Create a new mobile application that uses the API's.
- Only for design and development purposes.
- Make sure there is human in loop to validate the results.

## Pre-requisites

- Azure account
- Azure Open AI account
- GPt 4 Turbo model and create a deployment

## Steps

- Log into Azure Open AI studio
- Make sure you have API information ready in hard
- Now create the System prompt

```
API Details:
http://localhost:5001/customer API has all the information needed for customer information including shipping address and contact information
http://localhost:5001/Orders API has order management microservices, customer orders are stored and used to lookup order tracking information
http://localhost:5001/Shippping API has all different types of shipping information used for customer's to ship products for example UPS, Fedex, other shipping carrier information.
http://localhost:5001/products API has all the product information and details with inventory level to order items. Please only allow orders to process if inventory is more than 0
http://localhost:5001/logging API to log all information about the ordering process, shipping, products and other API relelated logs for debug purpose
http://localhost:5001/metadatastore API has all information to store in memory data to store for shopping cart and also cache information.

Only answer from the above API information provided. If can not be answer please respond not enough information to create code
```

- Save the System prompt
- Now ask the question

```
write code for python stream lit web app to build a online shopping cart and ability to check out?
```

- Model response

```
pip install streamlit  
```

```
import streamlit as st  
import requests  
  
# API Endpoints  
CUSTOMER_API = "http://localhost:5001/customer"  
ORDERS_API = "http://localhost:5001/Orders"  
SHIPPING_API = "http://localhost:5001/Shippping"  
PRODUCTS_API = "http://localhost:5001/products"  
LOGGING_API = "http://localhost:5001/logging"  
METADATASTORE_API = "http://localhost:5001/metadatastore"  
  
# Initialize the shopping cart in the session state if it doesn't exist  
if 'cart' not in st.session_state:  
    st.session_state['cart'] = {}  
  
# Add product to the cart  
def add_to_cart(product_id, quantity):  
    # Check if product exists and has inventory  
    product_response = requests.get(f"{PRODUCTS_API}/{product_id}")  
    product = product_response.json()  
    if product_response.status_code == 200 and product['inventory'] > 0:  
        st.session_state.cart[product_id] = st.session_state.cart.get(product_id, 0) + quantity  
        st.success('Product added to cart')  
    else:  
        st.error('Product not available or out of stock')  
  
# Checkout function  
def checkout():  
    customer_id = st.session_state.get('customer_id')  
    if not customer_id:  
        st.warning('Please login to continue')  
        return  
      
    if not st.session_state.cart:  
        st.warning('Your shopping cart is empty.')  
        return  
  
    # Create order  
    order_response = requests.post(ORDERS_API, json={  
        'customer_id': customer_id,  
        'order_items': st.session_state.cart  
    })  
  
    if order_response.status_code == 201:  
        order = order_response.json()  
        st.session_state.cart = {}  # Clear cart after checkout  
        st.success('Checkout successful!')  
        st.json(order)  
    else:  
        st.error('Checkout failed. Please try again.')  
  
# Streamlit layout  
st.title('Online Shopping Cart')  
  
# Display products (assuming a function to fetch and display products)  
products = requests.get(PRODUCTS_API).json()  
for product in products:  
    st.subheader(product['name'])  
    st.write(product['description'])  
    if product['inventory'] > 0:  
        quantity = st.number_input('Quantity', min_value=1, max_value=product['inventory'], key=product['id'])  
        st.button('Add to cart', on_click=add_to_cart, args=(product['id'], quantity))  
  
# Show cart  
st.write('Your shopping cart:')  
st.json(st.session_state.cart)  
  
# Checkout button  
if st.button('Checkout'):  
    checkout()  
  
# Run the Streamlit app  
if __name__ == '__main__':  
    st.run()  
```

```
#To run the Streamlit app, use the following command:

streamlit run streamlit_app.py 
```

```
This will start a local server and open your web browser to the app. The code above creates a simple Streamlit app that:
Initializes a shopping cart in the Streamlit session state.
Defines a function add_to_cart to add products to the cart, checking inventory through the PRODUCTS_API.
Defines a checkout function that uses the ORDERS_API to create an order and clear the cart upon success.
Lays out the app using Streamlit widgets to display products, add items to the cart, show the cart contents, and provide a checkout button.
```