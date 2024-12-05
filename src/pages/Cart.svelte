<script>
    export let products = [];
    export let total = 0;
    
    let newProduct = {
        name: '',
        costPrice: 0,
        sellingPrice: 0,
        quantity: 0,
        totalCost: 0
    };
    
    let editingIndex = -1;
    let editingProduct = null;
    let isPricePerUnit = true;
    let currentCostValue = 0;

    // Update values when cost input changes
    function handleCostChange(value) {
        currentCostValue = Number(value);
        if (isPricePerUnit) {
            newProduct.costPrice = currentCostValue;
            newProduct.totalCost = currentCostValue * newProduct.quantity;
        } else {
            newProduct.totalCost = currentCostValue;
            newProduct.costPrice = newProduct.quantity > 0 ? currentCostValue / newProduct.quantity : 0;
        }
    }

    // Update costs when quantity changes
    $: {
        if (isPricePerUnit) {
            newProduct.totalCost = newProduct.costPrice * newProduct.quantity;
        } else {
            newProduct.costPrice = newProduct.quantity > 0 ? newProduct.totalCost / newProduct.quantity : 0;
        }
    }

    function togglePriceMode() {
        isPricePerUnit = !isPricePerUnit;
        currentCostValue = isPricePerUnit ? newProduct.costPrice : newProduct.totalCost;
    }
    
    function addProduct() {
        if (newProduct.name && newProduct.sellingPrice >= newProduct.costPrice && newProduct.quantity > 0) {
            products = [...products, { 
                ...newProduct,
                costPrice: Number(newProduct.costPrice),
                totalCost: Number(newProduct.totalCost),
                profit: (newProduct.sellingPrice - newProduct.costPrice) * newProduct.quantity
            }];
            resetForm();
        }
    }
    
    function startEditing(index) {
        editingIndex = index;
        editingProduct = { ...products[index] };
    }
    
    function saveEdit() {
        if (editingProduct && editingIndex >= 0) {
            editingProduct.profit = (editingProduct.sellingPrice - editingProduct.costPrice) * editingProduct.quantity;
            products = products.map((p, i) => 
                i === editingIndex ? { ...editingProduct } : p
            );
            cancelEdit();
        }
    }
    
    function cancelEdit() {
        editingIndex = -1;
        editingProduct = null;
    }
    
    function deleteProduct(index) {
        products = products.filter((_, i) => i !== index);
    }
    
    function resetForm() {
        newProduct = {
            name: '',
            costPrice: 0,
            sellingPrice: 0,
            quantity: 0,
            totalCost: 0
        };
        currentCostValue = 0;
    }

    $: {
        total = products.reduce((sum, product) => 
            sum + (product.sellingPrice * product.quantity), 0);
    }

    $: totalCost = products.reduce((sum, product) => 
        sum + (product.costPrice * product.quantity), 0);

    $: totalProfit = products.reduce((sum, product) => 
        sum + ((product.sellingPrice - product.costPrice) * product.quantity), 0);
</script>

<div class="cart-container">
    <section class="add-product-section">
        <div class="add-product-form">
            <h2>Add New Product</h2>
            
            <div class="form-group">
                <label>
                    Product Name:
                    <input 
                        type="text" 
                        bind:value={newProduct.name}
                        placeholder="Enter product name"
                    >
                </label>
            </div>
            
            <div class="cost-price-toggle">
                <button 
                    class="toggle-btn" 
                    class:active={isPricePerUnit} 
                    on:click={togglePriceMode}
                >
                    Cost Per Unit
                </button>
                <button 
                    class="toggle-btn" 
                    class:active={!isPricePerUnit} 
                    on:click={togglePriceMode}
                >
                    Total Cost
                </button>
            </div>

            <div class="form-group">
                <label>
                    {isPricePerUnit ? 'Cost Price (per unit):' : 'Total Cost (all units):'}
                    <input 
                        type="number" 
                        bind:value={currentCostValue}
                        on:input={(e) => handleCostChange(e.target.value)}
                        min="0" 
                        step="0.01"
                    >
                </label>
            </div>

            <div class="form-group">
                <label>
                    Quantity:
                    <input type="number" bind:value={newProduct.quantity} min="0">
                </label>
            </div>

            {#if newProduct.quantity > 0}
                <div class="cost-summary">
                    {#if isPricePerUnit}
                        <p>Total Cost: ₹{newProduct.totalCost.toFixed(2)}</p>
                    {:else}
                        <p>Cost Per Unit: {newProduct.costPrice.toFixed(2)}</p>
                    {/if}
                </div>
            {/if}

            <div class="form-group">
                <label>
                    Selling Price (per unit):
                    <input type="number" bind:value={newProduct.sellingPrice} min="0" step="0.01">
                </label>
            </div>
            
            <button class="add-btn" on:click={addProduct}>
                <i class="fas fa-plus"></i>
                Add Product
            </button>
        </div>
    </section>

    <section class="products-section">
        <h2>Products in Cart</h2>
        {#if products.length === 0}
            <p class="empty-message">No products in cart</p>
        {:else}
            {#each products as product, index}
                <div class="product-item">
                    {#if editingIndex === index}
                        <div class="edit-form">
                            <div class="form-group">
                                <label>
                                    Name:
                                    <input type="text" bind:value={editingProduct.name}>
                                </label>
                            </div>
                            <div class="form-group">
                                <label>
                                    Cost Price:
                                    <input type="number" bind:value={editingProduct.costPrice} min="0" step="0.01">
                                </label>
                            </div>
                            <div class="form-group">
                                <label>
                                    Selling Price:
                                    <input type="number" bind:value={editingProduct.sellingPrice} min="0" step="0.01">
                                </label>
                            </div>
                            <div class="form-group">
                                <label>
                                    Quantity:
                                    <input type="number" bind:value={editingProduct.quantity} min="0">
                                </label>
                            </div>
                            <div class="edit-buttons">
                                <button class="save-btn" on:click={saveEdit}>Save</button>
                                <button class="cancel-btn" on:click={cancelEdit}>Cancel</button>
                            </div>
                        </div>
                    {:else}
                        <div class="product-details">
                            <h3>{product.name}</h3>
                            <p>Cost Price: ₹{product.costPrice.toFixed(2)}</p>
                            <p>Selling Price: ₹{product.sellingPrice.toFixed(2)}</p>
                            <p>Quantity: {product.quantity}</p>
                            <p>Total Cost: ₹{(product.costPrice * product.quantity).toFixed(2)}</p>
                            <p>Total Revenue: ₹{(product.sellingPrice * product.quantity).toFixed(2)}</p>
                            <p>Profit: ₹{((product.sellingPrice - product.costPrice) * product.quantity).toFixed(2)}</p>
                        </div>
                        <div class="action-buttons">
                            <button class="edit-btn" on:click={() => startEditing(index)} title="Edit">
                                <i class="fas fa-pencil-alt"></i>
                            </button>
                            <button class="delete-btn" on:click={() => deleteProduct(index)} title="Delete">
                                <i class="fas fa-trash-alt"></i>
                            </button>
                        </div>
                    {/if}
                </div>
            {/each}
        {/if}
    </section>

    <section class="summary-section">
        <div class="summary-cards">
            <div class="summary-card">
                <h3>Total Cost</h3>
                <p>₹{totalCost.toFixed(2)}</p>
            </div>
            <div class="summary-card">
                <h3>Total Revenue</h3>
                <p>₹{total.toFixed(2)}</p>
            </div>
            <div class="summary-card">
                <h3>Total Profit</h3>
                <p>₹{totalProfit.toFixed(2)}</p>
            </div>
        </div>
    </section>
</div>

<style>
    .cart-container {
        display: grid;
        gap: 2em;
    }

    section {
        background: white;
        border-radius: 12px;
        padding: 2em;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }

    h2 {
        color: #ff5e28;
        font-size: 1.8em;
        font-weight: 300;
        margin-bottom: 1.5em;
        text-align: center;
    }

    .form-group {
        margin: 1.5em 0;
    }

    label {
        display: block;
        margin-bottom: 0.8em;
        color: #444;
        font-weight: 500;
    }

    input {
        width: 100%;
        padding: 0.8em;
        border: 2px solid #ddd;
        border-radius: 8px;
        font-size: 1em;
        transition: border-color 0.3s ease;
    }

    input:focus {
        border-color: #ff5e28;
        outline: none;
    }

    .cost-price-toggle {
        display: flex;
        gap: 1em;
        margin: 1.5em 0;
    }

    .toggle-btn {
        flex: 1;
        padding: 1em;
        font-size: 1em;
        border: 2px solid #ddd;
        border-radius: 8px;
        background: #f5f5f5;
        cursor: pointer;
        transition: all 0.3s ease;
    }

    .toggle-btn.active {
        background: #ff5e28;
        border-color: #ff5e28;
    }

    .product-item {
        background: #f9f9f9;
        border-radius: 8px;
        padding: 1.5em;
        margin: 1em 0;
        display: flex;
        justify-content: space-between;
        align-items: center;
        transition: transform 0.3s ease;
    }

    .product-item:hover {
        transform: translateX(5px);
    }

    .action-buttons {
        display: flex;
        gap: 1em;
    }

    button {
        padding: 0.8em 1.5em;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: 500;
        transition: all 0.3s ease;
        background-color: #ff5e28;
    }

    button:hover {
        background-color: #ff7f50;
    }

    .summary-cards {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 1.5em;
    }

    .summary-card {
        background: #f9f9f9;
        padding: 1.5em;
        border-radius: 8px;
        text-align: center;
        transition: transform 0.3s ease;
    }

    .summary-card:hover {
        transform: translateY(-5px);
    }

    @media (max-width: 768px) {
        .cart-container {
            gap: 1em;
        }

        section {
            padding: 1em;
        }

        .summary-cards {
            grid-template-columns: 1fr;
        }
    }

    .edit-btn {
        background-color: #ff7f50;  /* Add back the background color */
        padding: 0.8em;
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .edit-btn:hover {
        background-color: #ffa07a;
    }

    .edit-btn i {
        font-size: 1.1em;
    }

    .save-btn {
        background-color: #4CAF50;  /* Keep success button green */
    }

    .delete-btn {
        background-color: #dc3545;  /* Keep delete button red */
        padding: 0.8em;  /* Make it square */
        width: 40px;     /* Fixed width */
        height: 40px;    /* Fixed height */
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .delete-btn i {
        font-size: 1.1em;
    }

    .summary-card h3 {
        color: #ff5e28;
    }

    .add-btn {
        width: 100%;
        padding: 1em;
        font-size: 1.1em;
        background-color: #ff5e28;
        color: white;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.5em;
        margin-top: 2em;
    }

    .add-btn:hover {
        background-color: #ff7f50;
        transform: translateY(-2px);
    }

    .add-btn:active {
        transform: translateY(0);
    }

    .add-btn i {
        font-size: 0.9em;
    }
</style> 