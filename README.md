## What is the purpose of this code?
The purpose of this code is to add a hidden input field to the product page. This can be used for many things, but in our case, it’s used to get a custom price from an external API and then use that as the price on checkout.

First, for testing purposes, we add a price in the hidden input field as you don’t give the code that calculates the price:

```php
<?php
add_action( 'woocommerce_before_add_to_cart_button', 'cxc_custom_hidden_product_field', 11 );
function cxc_custom_hidden_product_field() {
	echo '<input type="hidden" id="hidden_field" name="custom_price" class="custom_price" value="20">'; // Price is 20 for testing
}
?>
```

Then you will use the following to change the cart item price (WC_Session is not needed). The code goes into the function.php file of your active child theme (or active theme). 

```php
<?php
add_filter( 'woocommerce_add_cart_item_data', 'cxc_save_custom_fields_data_to_cart', 10, 2 );
function cxc_save_custom_fields_data_to_cart( $cart_item_data, $product_id ) {

	if( isset( $_POST['custom_price'] ) && ! empty( $_POST['custom_price'] )  ) {
        // Set the custom data in the cart item
		$cart_item_data['custom_price'] = (float) sanitize_text_field( $_POST['custom_price'] );

        // Make each item as a unique separated cart item
		$cart_item_data['unique_key'] = md5( microtime().rand() );
	}

	return $cart_item_data;
}

add_action( 'woocommerce_before_calculate_totals', 'cxc_change_cart_item_price', 99, 1 );
function cxc_change_cart_item_price( $cart ) {

	if ( ( is_admin() && ! defined( 'DOING_AJAX' ) ) ){  		
		return;
	}

	if ( did_action( 'woocommerce_before_calculate_totals' ) >= 2 ){
		return;
	}
    // Loop through cart items
	foreach ( $cart->get_cart() as $cart_item ) {
        // Set the new price  		
		if( isset($cart_item['custom_price']) ){
			$cart_item['data']->set_price( $cart_item['custom_price'] );
		}
	}
}
?>
```

Then this code in your existing theme’s functions.php. Now when you load product details page it will add a hidden text field above your add to cart button and then you just have to change the custom price on the textbox and after that when u use add to cart function it will add that custom price to your cart.
