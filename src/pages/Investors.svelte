<script>
    export let products = [];
    export let total = 0;
    export let investors = [];

    let newInvestor = {
        name: '',
        investment: 0,
        percentage: 0
    };

    let editingInvestorIndex = -1;
    let editingInvestor = null;

    $: totalCost = products.reduce((sum, product) => 
        sum + (product.costPrice * product.quantity), 0);

    $: totalProfit = products.reduce((sum, product) => 
        sum + ((product.sellingPrice - product.costPrice) * product.quantity), 0);

    function addInvestor() {
        if (newInvestor.name && newInvestor.investment > 0) {
            investors = [...investors, { ...newInvestor }];
            calculatePercentages();
            resetInvestorForm();
        }
    }

    function deleteInvestor(index) {
        investors = investors.filter((_, i) => i !== index);
        calculatePercentages();
    }

    function calculatePercentages() {
        const totalInvestment = investors.reduce((sum, inv) => sum + inv.investment, 0);
        investors = investors.map(inv => ({
            ...inv,
            percentage: (inv.investment / totalInvestment) * 100,
            profitShare: (inv.investment / totalInvestment) * totalProfit
        }));
    }

    function resetInvestorForm() {
        newInvestor = {
            name: '',
            investment: 0,
            percentage: 0
        };
    }

    $: {
        if (investors.length > 0) {
            calculatePercentages();
        }
    }

    $: totalInvestment = investors.reduce((sum, inv) => sum + inv.investment, 0);

    function startEditingInvestor(index) {
        editingInvestorIndex = index;
        editingInvestor = { ...investors[index] };
    }

    function saveInvestorEdit() {
        if (editingInvestor && editingInvestorIndex >= 0) {
            investors = investors.map((inv, i) => 
                i === editingInvestorIndex ? { ...editingInvestor } : inv
            );
            calculatePercentages();
            cancelInvestorEdit();
        }
    }

    function cancelInvestorEdit() {
        editingInvestorIndex = -1;
        editingInvestor = null;
    }
</script>

<div class="investors-container">
    <section class="overview-section">
        <h2>Investment Overview</h2>
        <div class="summary-section">
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
            <div class="summary-card">
                <h3>Total Investment</h3>
                <p>₹{totalInvestment.toFixed(2)}</p>
            </div>
        </div>
    </section>

    <section class="add-investor-section">
        <div class="add-investor-form">
            <h3>Add New Investor</h3>
            <div class="form-group">
                <label>
                    Investor Name:
                    <input 
                        type="text" 
                        bind:value={newInvestor.name}
                        placeholder="Enter investor name"
                        class="investor-input"
                    >
                </label>
            </div>
            
            <div class="form-group">
                <label>
                    Investment Amount:
                    <input 
                        type="number" 
                        bind:value={newInvestor.investment}
                        min="0" 
                        step="0.01"
                        placeholder="Enter investment amount"
                        class="investor-input"
                    >
                </label>
            </div>
            
            <button class="add-btn" on:click={addInvestor}>
                <i class="fas fa-plus"></i>
                Add Investor
            </button>
        </div>
    </section>

    <section class="investors-list-section">
        <h3>Investors</h3>
        {#if investors.length === 0}
            <p class="empty-message">No investors added yet</p>
        {:else}
            {#each investors as investor, index}
                <div class="investor-item">
                    {#if editingInvestorIndex === index}
                        <div class="edit-form">
                            <div class="form-group">
                                <label>
                                    Name:
                                    <input 
                                        type="text" 
                                        bind:value={editingInvestor.name}
                                        class="investor-input"
                                    >
                                </label>
                            </div>
                            <div class="form-group">
                                <label>
                                    Investment Amount:
                                    <input 
                                        type="number" 
                                        bind:value={editingInvestor.investment}
                                        min="0" 
                                        step="0.01"
                                        class="investor-input"
                                    >
                                </label>
                            </div>
                            <div class="edit-buttons">
                                <button class="save-btn" on:click={saveInvestorEdit}>
                                    <i class="fas fa-check"></i>
                                </button>
                                <button class="cancel-btn" on:click={cancelInvestorEdit}>
                                    <i class="fas fa-times"></i>
                                </button>
                            </div>
                        </div>
                    {:else}
                        <div class="investor-details">
                            <h4>{investor.name}</h4>
                            <p>Investment: ₹{investor.investment.toFixed(2)}</p>
                            <p>Share: {investor.percentage.toFixed(2)}%</p>
                            <p>Profit Share: ₹{investor.profitShare.toFixed(2)}</p>
                        </div>
                        <div class="action-buttons">
                            <button class="edit-btn" on:click={() => startEditingInvestor(index)} title="Edit">
                                <i class="fas fa-pencil-alt"></i>
                            </button>
                            <button class="delete-btn" on:click={() => deleteInvestor(index)} title="Delete">
                                <i class="fas fa-trash-alt"></i>
                            </button>
                        </div>
                    {/if}
                </div>
            {/each}
        {/if}
    </section>

    <section class="products-summary-section">
        <h3>Products Summary</h3>
        <table>
            <thead>
                <tr>
                    <th>Product</th>
                    <th>Quantity</th>
                    <th>Cost Price</th>
                    <th>Selling Price</th>
                    <th>Total Cost</th>
                    <th>Total Revenue</th>
                    <th>Profit</th>
                </tr>
            </thead>
            <tbody>
                {#each products as product}
                    <tr>
                        <td>{product.name}</td>
                        <td>{product.quantity}</td>
                        <td>₹{product.costPrice.toFixed(2)}</td>
                        <td>₹{product.sellingPrice.toFixed(2)}</td>
                        <td>₹{(product.costPrice * product.quantity).toFixed(2)}</td>
                        <td>₹{(product.sellingPrice * product.quantity).toFixed(2)}</td>
                        <td>₹{((product.sellingPrice - product.costPrice) * product.quantity).toFixed(2)}</td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </section>
</div>

<style>
    .investors-container {
        display: grid;
        gap: 2em;
    }

    section {
        background: white;
        border-radius: 12px;
        padding: 2em;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }

    h2, h3 {
        color: #ff5e28;
        font-weight: 300;
        margin-bottom: 1.5em;
        text-align: center;
    }

    h2 {
        font-size: 1.8em;
    }

    h3 {
        font-size: 1.5em;
    }

    .summary-section {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 1.5em;
        margin-bottom: 2em;
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

    table {
        width: 100%;
        border-collapse: separate;
        border-spacing: 0;
        margin-top: 1em;
        font-size: 0.9em;
    }

    th, td {
        padding: 0.8em;
        text-align: right;
        border-bottom: 1px solid #eee;
    }

    th:first-child, 
    td:first-child {
        text-align: left;
    }

    th {
        background: #f9f9f9;
        color: #ff5e28;
        font-weight: 500;
        white-space: nowrap;
    }

    tr:hover td {
        background: #f5f5f5;
    }

    .investor-item {
        background: white;
        padding: 1.5em;
        margin: 1em 0;
        border-radius: 8px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        transition: all 0.3s ease;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    }

    .investor-item:hover {
        transform: translateX(5px);
    }

    @media (max-width: 768px) {
        .investors-container {
            gap: 1em;
        }

        section {
            padding: 1em;
        }

        .summary-section {
            grid-template-columns: 1fr;
        }

        table {
            display: block;
            overflow-x: auto;
        }
    }

    @media (max-width: 1024px) {
        .products-summary-section {
            overflow-x: auto;
        }

        table {
            min-width: 800px;
        }
    }

    button {
        background-color: #ff5e28;
    }

    button:hover {
        background-color: #ff7f50;
    }

    .delete-btn {
        background-color: #dc3545;  /* Keep delete button red */
    }

    .delete-btn:hover {
        background-color: #c82333;
    }

    .delete-btn {
        padding: 0.8em;
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .delete-btn i {
        font-size: 1.1em;
    }

    .add-investor-form {
        background: #f9f9f9;
        padding: 2em;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    }

    .form-group {
        margin: 1.5em 0;
    }

    label {
        display: block;
        margin-bottom: 0.8em;
        color: #444;
        font-weight: 500;
        font-size: 1.1em;
    }

    .investor-input {
        width: 100%;
        padding: 0.8em;
        border: 2px solid #ddd;
        border-radius: 8px;
        font-size: 1em;
        transition: all 0.3s ease;
        background: white;
    }

    .investor-input:focus {
        border-color: #ff5e28;
        outline: none;
        box-shadow: 0 0 0 3px rgba(255, 94, 40, 0.1);
    }

    .investor-input::placeholder {
        color: #aaa;
    }

    .add-btn {
        width: 100%;
        padding: 1em;
        font-size: 1.1em;
        background-color: #ff5e28;
        color: white;
        border: none;
        border-radius: 6px;
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

    .investor-item {
        background: white;
        padding: 1.5em;
        margin: 1em 0;
        border-radius: 8px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        transition: all 0.3s ease;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    }

    .investor-details {
        flex: 1;
    }

    .investor-details h4 {
        color: #333;
        font-size: 1.2em;
        margin: 0 0 0.5em 0;
    }

    .investor-details p {
        color: #666;
        margin: 0.3em 0;
        font-size: 1em;
    }

    @media (max-width: 768px) {
        .add-investor-form {
            padding: 1.5em;
        }

        .investor-item {
            padding: 1.2em;
        }

        .add-btn {
            padding: 0.8em;
        }
    }

    .edit-form {
        width: 100%;
        padding: 1em;
    }

    .edit-buttons {
        display: flex;
        gap: 0.5em;
        justify-content: flex-end;
        margin-top: 1em;
    }

    .action-buttons {
        display: flex;
        gap: 0.5em;
    }

    .edit-btn {
        background-color: #ff7f50;
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

    .save-btn {
        background-color: #4CAF50;
        padding: 0.8em;
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .save-btn:hover {
        background-color: #45a049;
    }

    .cancel-btn {
        background-color: #6c757d;
        padding: 0.8em;
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .cancel-btn:hover {
        background-color: #5a6268;
    }

    .edit-form .form-group {
        margin: 0.5em 0;
    }

    .edit-form input {
        width: 100%;
        padding: 0.5em;
        border: 2px solid #ddd;
        border-radius: 6px;
        font-size: 1em;
    }

    .edit-form input:focus {
        border-color: #ff5e28;
        outline: none;
        box-shadow: 0 0 0 3px rgba(255, 94, 40, 0.1);
    }

    /* Update the border-radius for all action buttons */
    .edit-btn, .delete-btn, .save-btn, .cancel-btn {
        border-radius: 6px;  /* Match Cart tab's border-radius */
    }

    /* If you want to update the add button too */
    .add-btn {
        border-radius: 6px;  /* Match Cart tab's border-radius */
    }
</style> 