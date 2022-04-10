# oBundle - BigCommerce Web Developer Test



### Link
Link to the Special Items page: <strong>https://obundle-test.mybigcommerce.com/special-items/</strong>
Preview Code: <strong>dcajajdqoi</strong>

### Overview of Tasks
1.  Create a product called Special Item which will be assigned to a new category called Special Items. Be sure to add at least 2 images during the product creation.
    
2.  The Special Item should be the only item which shows in this category - create a feature that will show the product's second image when it is hovered on. 


<strong>Updated code in card.html to implement task 2</strong>
```
// card.html

<div class="card-img-container">
    <img class="card-image lazyload" 
        data-sizes="auto" 
        src="{{cdn 'img/loading.svg'}}" 
        alt="{{image.alt}}" 
        title="{{image.alt}}"
        data-src="{{getImage image}}"
        data-hoverimage="{{#replace '{:size}' images.1.data}}500x659{{/replace}}{{getImage images.1.data 'productgallery_size' (cdn theme_settings.default_image_product)}}"
    >
</div> 

<script>
$('.card-figure').hover(
    	function() {
          var image = $(this).find('.card-image');
          var newImg = image.attr('data-hoverimage');

          var currentImg = image.attr('src');
    		if (newImg && newImg != '') image.attr('src', newImg);
       }, function() {
          var image = $(this).find('.card-image');
          var newImg = image.attr('data-src');
          var currentImg = image.attr('src');
    		if (newImg && newImg != '') image.attr('src', newImg);
    	}
    );
</script>
```

3.  Add a button at the top of the category page labeled Add All To Cart. When clicked, the product will be added to the cart. Notify the user that the product has been added. 
4. If the cart has an item in it - show a button next to the Add All To Cart button which says Remove All Items. When clicked it should clear the cart and notify the user. 
   
<em>NOTE: Both buttons should utilize the Storefront API for completion. </em>


<strong>Updated code in product-listing.html to implement steps 3 and 4</strong>
```
// product-listing.html


// Add Buttons for adding all items
<div class="button-wrapper">
    <button class="page-add-all-items" onClick='addAllItemsToCart()' data-reveal-id="modal-added">
        Add All To Cart 
    </button>

    <button class="page-remove-all-items display-none" onClick='removeAllItemsInCart()' data-reveal-id="modal-removed">
        Remove All Items
    </button>
</div>

// Simple Modal Popups
<div id="modal-added" class="modal modal--small" role="alert" data-reveal>
    <a href="#" class="modal-close" aria-label="{{lang 'common.close'}}" role="button">
        <span aria-hidden="true">&#215;</span>
    </a>
    <div class="modal-content modal-content-custom">
        <h2>Item(s) Added To Cart</h2>
    </div>
</div>

<div id="modal-removed" class="modal modal--small" role="alert" data-reveal>
    <a href="#" class="modal-close" aria-label="{{lang 'common.close'}}" role="button">
        <span aria-hidden="true">&#215;</span>
    </a>
    <div class="modal-content modal-content-custom">
        <h2>Item(s) Removed From Cart</h2>
    </div>   
</div>


<script>

    // Check if items are in cart, they are, show remove all items button
    $(document).ready(()=> {

        const handleCartHasItem = async () => {
            let res = await getCart('/api/storefront/carts');
    
            if (!res) {
                $('.page-remove-all-items').addClass('display-none');
            } else {
                $('.page-remove-all-items').removeClass('display-none');
            }
        }
    
        handleCartHasItem();
    }); 


    // handle add all items to cart
    const addAllItemsToCart = async () => {
        const jsContext = JSON.parse({{jsContext}});
        const { products, cartId } = jsContext;



        let data = {
            lineItems: products.map(product => ({
                quantity: 1,
                productId: product.id
            }))
        }

        try {
            const res = await createCart(data);
            $('.page-remove-all-items').removeClass('display-none');
            location.reload();
            alert('All Items Added');
        } catch (err) {
            console.errror(err)
        }
      
    };

    // utilize storefront API to create cart and add items in
    const createCart = async (cartItems) => {

        try {
            const res = await fetch('/api/storefront/carts', {
                method: "POST",
                credentials: "same-origin",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify(cartItems),
            })
    
            return await res.json();
        } catch (err) {
            console.error(err);
        }
    };

    
    // handle remove all items from cart
    const removeAllItemsInCart = async () => {
        let productIds = [];
        
        const jsContext = JSON.parse({{jsContext}});
        const { products, cartId } = jsContext;

        if (!cartId) return;
        
        try {
            let res = await getCart('/api/storefront/carts');

            res.lineItems.physicalItems.forEach(lineItem => {
                productIds.push(lineItem.id)
            })
            
            productIds.forEach(productId => {
                const res =  deleteCartItem('/api/storefront/carts/', cartId, productId); 
            })
            $('.page-remove-all-items').addClass('display-none');
            location.reload();
            alert('All Items Removed');
        } catch (err) {
            console.error(err);
            return;
        }
    }
    
    // utilize storefront API to get items in cart
    const getCart = async (url) => {
        const res = await fetch(url, {
            method: "GET",
            credentials: "same-origin"
        })
        const data = await res.json();
        return data[0];
    };

    // utilize storefront API to delete all items in cart
    const deleteCartItem = async (url, cartId, itemId) => {
        return await fetch("/api/storefront/carts/" + cartId + "/items/" + itemId, {
            method: "DELETE",
            credentials: "same-origin",
            headers: {
                "Content-Type": "application/json",
            },
        });
    };
  


  </script>
```



