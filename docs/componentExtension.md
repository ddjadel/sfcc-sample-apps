## Customizing and Extending Components

As mentioned in the Architecture.md when customizing or extending the sample app component, do not modify the packages within `@sfcc-bff` and `@sfcc-core`. These packages will be published and consumed via NPM. Instead, create a new custom package within the monorepo that registers itself with `@sfcc-core` and provides access to data from a third-party service.

This example below illustates how to create a product recommendations extension for product details component:

Background : Product Details Page displays product name, item id, color swatches, images, price etc, our goal is extend this component to display this product's recommendations on Product Detail Page. 

1. In the storefront-lwc, create a new extension called productDetailExtension.mjs, in there register this extension with core using this key : API_EXTENSION_KEY. Since extensions can have multiple entries per extension key, we can simply register a new extension with the same key: 

`core.registerExtension(API_EXTENSIONS_KEY, function (config) {
    const extensions = new ProductDetailExtensions();
    return extensions;
});`

2. Define the Product extendion is recommendation, then define the Recommendation TypeDef 

`const productRecommendationTypeDef = gql
    type Recommendation {
        productId: String
        productName: String
        image: Image
    }
    extend type Product {
        recommendations: [Recommendation]
    }
;` 

3. Resolve the Recommendation for a Product
`const productRecommendationResolver = (config) => {
    return {
        Product: {
            recommendations: async (product) => {
                if (product.recommendations) {
                    return Promise.all(product.recommendations.map( async recommendation => {
                        const apiProduct = await getClientProduct(config, recommendation.recommendedItemId);
                            return {productId: apiProduct.id, productName:apiProduct.name, image: new Image(apiProduct.imageGroups[2].images[0])};
                        })
                    )
                }
            }
        }
    }
}`

4. Add the product recommendation extension to the BFF by import this extension created in step 1 to the sample-app.mjs file. `import './extension/productDetailExtension';`

5. In the productdetailadapator file we need to tell the BFF the data we want to query for product recommendations.
`recommendations {
                    productId
                    productName
                    image {
                        title
                        link
                        alt
                        }
                }`

6. In the productdetail.html, we can consume the recommendataions data returned from the BFF.
` <!-- Product Recommendations -->
<template if:true={product.recommendations}>
    <div class="recommendation">
        <template for:each={product.recommendations} for:item="recommendation">
            <div class='col-6 col-sm-4 grid-gutter' key={recommendation.productId} >
                <div class='product' >
                    <commerce-product-tile product={recommendation}></commerce-product-tile>
                </div>
            </div>
        </template>
    </div>
</template>`

