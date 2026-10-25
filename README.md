# khan

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>Alam Tiles - Complete Inventory Management System</title>
    <!-- Tailwind CSS + FontAwesome Icons + Google Fonts -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,300;14..32,400;14..32,500;14..32,600;14..32,700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <!-- React & ReactDOM -->
    <script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.development.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.development.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/babel-standalone@6.26.0/babel.min.js"></script>
    <style>
        body { font-family: 'Inter', sans-serif; background: #f4f7fc; }
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #e2e8f0; border-radius: 10px; }
        ::-webkit-scrollbar-thumb { background: #94a3b8; border-radius: 10px; }
        .sidebar-item.active { background: #1e40af; color: white; box-shadow: 0 2px 6px rgba(0,0,0,0.1); }
        .card-hover:hover { box-shadow: 0 10px 20px rgba(0,0,0,0.05); transform: translateY(-2px); transition: all 0.2s; }
        input, select, textarea { transition: all 0.2s; }
        input:focus, select:focus { border-color: #2563eb; outline: none; box-shadow: 0 0 0 2px rgba(37,99,235,0.2); }
        .invoice-row:hover { background-color: #fef9c3; }
    </style>
</head>
<body class="bg-gray-100">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // ---------- UTILS & LOCALSTORAGE KEYS ----------
        const STORAGE_KEYS = {
            PRODUCTS: 'tile_products',
            CUSTOMERS: 'tile_customers',
            SUPPLIERS: 'tile_suppliers',
            SALES: 'tile_sales',
            PURCHASES: 'tile_purchases',
            CATEGORIES: 'tile_categories',
            COSTS: 'tile_costs',
            CASH_TRANSACTIONS: 'tile_cash_trans',
            COMPANIES: 'tile_companies'
        };

        // Initial seed data (based on screenshots)
        const getInitialProducts = () => [
            { id: 'p1', name: 'TW 6612 DP', category: 'floor', company: 'kongfu ceramic', sqftPerPiece: 2.25, pcsPerCarton: 4, stockPieces: 48, purchasePricePerPiece: 85, sellPricePerPiece: 120 },
            { id: 'p2', name: 'TW 206 D', category: 'wall', company: 'kongfu ceramic', sqftPerPiece: 0.67, pcsPerCarton: 25, stockPieces: 275, purchasePricePerPiece: 22, sellPricePerPiece: 35 },
            { id: 'p3', name: 'BLP 6106', category: 'floor', company: 'kongfu ceramic', sqftPerPiece: 4.0, pcsPerCarton: 3, stockPieces: 12, purchasePricePerPiece: 400, sellPricePerPiece: 520 },
            { id: 'p4', name: 'FR 2000R', category: 'wall', company: 'Fress ceramics', sqftPerPiece: 0.67, pcsPerCarton: 25, stockPieces: 220, purchasePricePerPiece: 24, sellPricePerPiece: 38 },
            { id: 'p5', name: 'FW 400', category: 'floor', company: 'Fress ceramics', sqftPerPiece: 1.78, pcsPerCarton: 9, stockPieces: 225, purchasePricePerPiece: 95, sellPricePerPiece: 140 },
            { id: 'p6', name: 'KF 1100', category: 'wall', company: 'kongfu ceramic', sqftPerPiece: 0.67, pcsPerCarton: 25, stockPieces: 500, purchasePricePerPiece: 21, sellPricePerPiece: 32 }
        ];
        const getInitialCustomers = () => [
            { id: 'c1', name: 'MAK', mobile: '01686171669', address: 'মতিঝিল, ঢাকা', dueAmount: 15520 },
            { id: 'c2', name: 'সুনীল মন্ডল', mobile: '01711223344', address: 'মিরপুর', dueAmount: 2020 },
            { id: 'c3', name: 'রহিম ট্রেডার্স', mobile: '01988776655', address: 'ঈগল মোড়', dueAmount: 0 }
        ];
        const getInitialSuppliers = () => ([
            { id: 's1', name: 'Kongfu Ceramics Ltd', mobile: '01811111111', address: 'Narayanganj', advance: 0, totalDue: 0 },
            { id: 's2', name: 'Fress Ceramics', mobile: '01722222222', address: 'Savar', advance: 0, totalDue: 0 }
        ]);
        const getInitialCategories = () => ([
            { id: 'cat1', name: 'floor' }, { id: 'cat2', name: 'wall' }, { id: 'cat3', name: 'viksol' }, { id: 'cat4', name: 'Tile Spacers' }
        ]);
        const getInitialCompanies = () => ([
            { id: 'comp1', name: 'kongfu ceramic' }, { id: 'comp2', name: 'Fress ceramics' }, { id: 'comp3', name: 'Tile Spacer' }
        ]);

        // helper to load/save
        const loadData = (key, initialFn) => { const stored = localStorage.getItem(key); return stored ? JSON.parse(stored) : initialFn(); };
        const saveData = (key, data) => localStorage.setItem(key, JSON.stringify(data));

        // ---------- MAIN APP ----------
        const App = () => {
            const [activeMenu, setActiveMenu] = useState('dashboard');
            // Global state
            const [products, setProducts] = useState([]);
            const [customers, setCustomers] = useState([]);
            const [suppliers, setSuppliers] = useState([]);
            const [sales, setSales] = useState([]);
            const [purchases, setPurchases] = useState([]);
            const [categories, setCategories] = useState([]);
            const [costs, setCosts] = useState([]);
            const [cashTransactions, setCashTransactions] = useState([]);
            const [companies, setCompanies] = useState([]);

            // Load all data on mount
            useEffect(() => {
                setProducts(loadData(STORAGE_KEYS.PRODUCTS, getInitialProducts));
                setCustomers(loadData(STORAGE_KEYS.CUSTOMERS, getInitialCustomers));
                setSuppliers(loadData(STORAGE_KEYS.SUPPLIERS, getInitialSuppliers));
                setSales(loadData(STORAGE_KEYS.SALES, () => []));
                setPurchases(loadData(STORAGE_KEYS.PURCHASES, () => []));
                setCategories(loadData(STORAGE_KEYS.CATEGORIES, getInitialCategories));
                setCosts(loadData(STORAGE_KEYS.COSTS, () => []));
                setCashTransactions(loadData(STORAGE_KEYS.CASH_TRANSACTIONS, () => []));
                setCompanies(loadData(STORAGE_KEYS.COMPANIES, getInitialCompanies));
            }, []);

            // persist whenever key state changes
            useEffect(() => { if(products.length) saveData(STORAGE_KEYS.PRODUCTS, products); }, [products]);
            useEffect(() => { if(customers.length) saveData(STORAGE_KEYS.CUSTOMERS, customers); }, [customers]);
            useEffect(() => { if(suppliers.length) saveData(STORAGE_KEYS.SUPPLIERS, suppliers); }, [suppliers]);
            useEffect(() => { if(sales.length) saveData(STORAGE_KEYS.SALES, sales); }, [sales]);
            useEffect(() => { if(purchases.length) saveData(STORAGE_KEYS.PURCHASES, purchases); }, [purchases]);
            useEffect(() => { if(categories.length) saveData(STORAGE_KEYS.CATEGORIES, categories); }, [categories]);
            useEffect(() => { if(costs.length) saveData(STORAGE_KEYS.COSTS, costs); }, [costs]);
            useEffect(() => { if(cashTransactions.length) saveData(STORAGE_KEYS.CASH_TRANSACTIONS, cashTransactions); }, [cashTransactions]);
            useEffect(() => { if(companies.length) saveData(STORAGE_KEYS.COMPANIES, companies); }, [companies]);

            // Sidebar menu items
            const menuItems = [
                { id: 'dashboard', label: 'Dashboard', icon: 'fas fa-tachometer-alt' },
                { id: 'saleForm', label: 'Sale Form / Invoice', icon: 'fas fa-file-invoice' },
                { id: 'saleHistory', label: 'Sale History', icon: 'fas fa-history' },
                { id: 'returnHistory', label: 'Return History', icon: 'fas fa-undo-alt' },
                { id: 'myStock', label: 'My Stock', icon: 'fas fa-boxes' },
                { id: 'godownStatus', label: 'Godown Status', icon: 'fas fa-warehouse' },
                { id: 'addCustomer', label: 'Add Customer & List', icon: 'fas fa-user-plus' },
                { id: 'customerReport', label: 'Customer Report', icon: 'fas fa-chart-line' },
                { id: 'addCategory', label: 'Add Category', icon: 'fas fa-tags' },
                { id: 'addProduct', label: 'Add Product', icon: 'fas fa-cube' },
                { id: 'purchaseProduct', label: 'Purchase Product', icon: 'fas fa-shopping-cart' },
                { id: 'purchaseHistory', label: 'Purchase History', icon: 'fas fa-truck' },
                { id: 'addSupplier', label: 'Add Supplier & List', icon: 'fas fa-truck-moving' },
                { id: 'supplierPayment', label: 'Supplier Payment', icon: 'fas fa-money-bill-wave' },
                { id: 'addCost', label: 'Add Cost', icon: 'fas fa-coins' },
                { id: 'costReport', label: 'Cost Report', icon: 'fas fa-chart-pie' },
                { id: 'cashInOut', label: 'Cash In/Out', icon: 'fas fa-exchange-alt' },
                { id: 'profit', label: 'Profit / Daily Account', icon: 'fas fa-chart-simple' }
            ];

            // Helper: update customer due after sale
            const updateCustomerDue = (customerId, amountPaid, totalInvoiceAmount) => {
                const updated = customers.map(c => {
                    if(c.id === customerId) {
                        const newDue = c.dueAmount + (totalInvoiceAmount - amountPaid);
                        return { ...c, dueAmount: newDue };
                    }
                    return c;
                });
                setCustomers(updated);
            };

            const addSaleInvoice = (invoiceData) => {
                // invoiceData: { customerId, items: [{productId, pieces, totalPrice}], totalAmount, paidAmount, dueAmount, date }
                // update stock
                let updatedProducts = [...products];
                for(let item of invoiceData.items) {
                    const prodIndex = updatedProducts.findIndex(p => p.id === item.productId);
                    if(prodIndex !== -1) {
                        const newPieces = updatedProducts[prodIndex].stockPieces - item.pieces;
                        if(newPieces < 0) alert(`Insufficient stock for ${updatedProducts[prodIndex].name}`);
                        else updatedProducts[prodIndex] = { ...updatedProducts[prodIndex], stockPieces: newPieces };
                    }
                }
                setProducts(updatedProducts);
                const newSale = { id: Date.now().toString(), ...invoiceData, invoiceNo: `INV-${Math.floor(Math.random()*9000)+1000}` };
                setSales(prev => [newSale, ...prev]);
                updateCustomerDue(invoiceData.customerId, invoiceData.paidAmount, invoiceData.totalAmount);
                // add cash transaction (if paid)
                if(invoiceData.paidAmount > 0) {
                    const cashTrans = { id: Date.now() + '_cash', type: 'in', amount: invoiceData.paidAmount, reason: `Sale ${newSale.invoiceNo}`, date: invoiceData.date };
                    setCashTransactions(prev => [cashTrans, ...prev]);
                }
            };

            // return product logic (simplified)
            const addReturn = (saleId, returnedAmount) => {
                setSales(prev => prev.map(s => s.id === saleId ? { ...s, isReturned: true, returnedAmount } : s));
                alert("Return recorded (adjust stock manually via purchase or edit)");
            };

            return (
                <div className="flex h-screen bg-gray-100 overflow-hidden">
                    {/* Sidebar */}
                    <div className="w-72 bg-white shadow-xl z-10 overflow-y-auto border-r border-gray-200">
                        <div className="p-5 bg-gradient-to-r from-blue-800 to-blue-700 text-white">
                            <h1 className="text-2xl font-bold flex items-center gap-2"><i className="fas fa-th-large"></i> Alam Tiles</h1>
                            <p className="text-xs text-blue-100 mt-1">Inventory Management</p>
                        </div>
                        <nav className="py-4 px-3 space-y-1">
                            {menuItems.map(item => (
                                <button key={item.id} onClick={() => setActiveMenu(item.id)} className={`sidebar-item w-full text-left px-4 py-2.5 rounded-xl flex items-center gap-3 text-sm font-medium transition-all ${activeMenu === item.id ? 'active bg-blue-600 text-white shadow-md' : 'text-gray-700 hover:bg-gray-100'}`}>
                                    <i className={`${item.icon} w-5 text-center`}></i> {item.label}
                                </button>
                            ))}
                        </nav>
                    </div>

                    {/* Main Content */}
                    <div className="flex-1 overflow-y-auto p-6">
                        {activeMenu === 'dashboard' && <Dashboard sales={sales} products={products} customers={customers} cashTransactions={cashTransactions} />}
                        {activeMenu === 'saleForm' && <SaleForm products={products} customers={customers} addSaleInvoice={addSaleInvoice} />}
                        {activeMenu === 'saleHistory' && <SaleHistory sales={sales} customers={customers} />}
                        {activeMenu === 'returnHistory' && <ReturnHistory sales={sales.filter(s => s.isReturned)} />}
                        {activeMenu === 'myStock' && <MyStock products={products} />}
                        {activeMenu === 'godownStatus' && <GodownStatus products={products} purchases={purchases} sales={sales} />}
                        {activeMenu === 'addCustomer' && <AddCustomer customers={customers} setCustomers={setCustomers} />}
                        {activeMenu === 'customerReport' && <CustomerReport customers={customers} sales={sales} />}
                        {activeMenu === 'addCategory' && <AddCategory categories={categories} setCategories={setCategories} />}
                        {activeMenu === 'addProduct' && <AddProduct products={products} setProducts={setProducts} categories={categories} companies={companies} />}
                        {activeMenu === 'purchaseProduct' && <PurchaseProduct products={products} setProducts={setProducts} suppliers={suppliers} setPurchases={setPurchases} purchases={purchases} />}
                        {activeMenu === 'purchaseHistory' && <PurchaseHistory purchases={purchases} suppliers={suppliers} />}
                        {activeMenu === 'addSupplier' && <AddSupplier suppliers={suppliers} setSuppliers={setSuppliers} />}
                        {activeMenu === 'supplierPayment' && <SupplierPayment suppliers={suppliers} setSuppliers={setSuppliers} cashTransactions={setCashTransactions} />}
                        {activeMenu === 'addCost' && <AddCost costs={costs} setCosts={setCosts} />}
                        {activeMenu === 'costReport' && <CostReport costs={costs} />}
                        {activeMenu === 'cashInOut' && <CashInOut cashTransactions={cashTransactions} setCashTransactions={setCashTransactions} />}
                        {activeMenu === 'profit' && <ProfitReport sales={sales} costs={costs} cashTransactions={cashTransactions} />}
                    </div>
                </div>
            );
        };

        // ----- DASHBOARD COMPONENT (Summary) -----
        const Dashboard = ({ sales, products, customers, cashTransactions }) => {
            const today = new Date().toISOString().slice(0,10);
            const todaySales = sales.filter(s => s.date === today).reduce((sum, s) => sum + s.totalAmount, 0);
            const totalStockValue = products.reduce((sum, p) => sum + (p.stockPieces * p.purchasePricePerPiece), 0);
            const totalDue = customers.reduce((sum, c) => sum + c.dueAmount, 0);
            return (
                <div className="space-y-6">
                    <h2 className="text-2xl font-bold text-gray-800"><i className="fas fa-chart-line mr-2 text-blue-600"></i> Dashboard</h2>
                    <div className="grid grid-cols-1 md:grid-cols-4 gap-5">
                        <div className="bg-white p-5 rounded-2xl shadow-sm border-l-8 border-blue-500"><p className="text-gray-500">Today's Sales</p><p className="text-3xl font-bold">৳{todaySales.toFixed(2)}</p></div>
                        <div className="bg-white p-5 rounded-2xl shadow-sm border-l-8 border-green-500"><p className="text-gray-500">Stock Value</p><p className="text-3xl font-bold">৳{totalStockValue.toFixed(2)}</p></div>
                        <div className="bg-white p-5 rounded-2xl shadow-sm border-l-8 border-red-500"><p className="text-gray-500">Total Receivables</p><p className="text-3xl font-bold">৳{totalDue.toFixed(2)}</p></div>
                        <div className="bg-white p-5 rounded-2xl shadow-sm border-l-8 border-purple-500"><p className="text-gray-500">Products SKU</p><p className="text-3xl font-bold">{products.length}</p></div>
                    </div>
                    <div className="bg-white p-5 rounded-2xl shadow-sm"><h3 className="font-semibold text-lg mb-2">Godown Quick Status</h3><p>📦 Low Stock Items: {products.filter(p => p.stockPieces < 30).map(p=>p.name).join(', ') || 'All sufficient'}</p></div>
                </div>
            );
        };

        // Sale Form with dynamic rows (simplified but full functional)
        const SaleForm = ({ products, customers, addSaleInvoice }) => {
            const [selectedCustomer, setSelectedCustomer] = useState('');
            const [rows, setRows] = useState([{ productId: '', pieces: 1, pricePerPiece: 0, total: 0 }]);
            const [paidAmount, setPaidAmount] = useState(0);
            const [date] = useState(new Date().toISOString().slice(0,10));
            const getProduct = (id) => products.find(p => p.id === id);
            const updateRow = (idx, field, value) => {
                const newRows = [...rows];
                if(field === 'productId') {
                    const prod = products.find(p => p.id === value);
                    newRows[idx].productId = value;
                    newRows[idx].pricePerPiece = prod ? prod.sellPricePerPiece : 0;
                    newRows[idx].pieces = 1;
                    newRows[idx].total = prod ? prod.sellPricePerPiece : 0;
                } else if(field === 'pieces') {
                    const pcs = parseInt(value) || 0;
                    newRows[idx].pieces = pcs;
                    newRows[idx].total = pcs * newRows[idx].pricePerPiece;
                }
                setRows(newRows);
            };
            const addRow = () => setRows([...rows, { productId: '', pieces: 1, pricePerPiece: 0, total: 0 }]);
            const totalAmount = rows.reduce((sum, r) => sum + r.total, 0);
            const submitInvoice = () => {
                if(!selectedCustomer) return alert('Select customer');
                const items = rows.filter(r => r.productId && r.pieces > 0).map(r => ({ productId: r.productId, pieces: r.pieces, totalPrice: r.total }));
                if(items.length===0) return alert('Add at least one product');
                const dueAmount = totalAmount - paidAmount;
                addSaleInvoice({ customerId: selectedCustomer, items, totalAmount: totalAmount, paidAmount: parseFloat(paidAmount)||0, dueAmount: dueAmount>0?dueAmount:0, date });
                alert('Invoice created successfully!');
                setRows([{ productId: '', pieces: 1, pricePerPiece: 0, total: 0 }]); setPaidAmount(0); setSelectedCustomer('');
            };
            return (
                <div className="bg-white p-6 rounded-2xl shadow-lg"><h2 className="text-2xl font-bold mb-4">🧾 New Invoice</h2>
                <div className="mb-4"><label>Customer</label><select value={selectedCustomer} onChange={e=>setSelectedCustomer(e.target.value)} className="w-full p-2 border rounded"><option value="">Select Customer</option>{customers.map(c=> <option key={c.id} value={c.id}>{c.name} ({c.mobile})</option>)}</select></div>
                <div className="overflow-x-auto"><table className="w-full"><thead><tr><th>Product</th><th>Pieces</th><th>Price/Piece</th><th>Total</th></tr></thead><tbody>{rows.map((row,i)=> <tr key={i}><td><select value={row.productId} onChange={e=>updateRow(i,'productId',e.target.value)} className="border p-1 rounded"><option value="">Select</option>{products.map(p=> <option key={p.id} value={p.id}>{p.name} (Stock: {p.stockPieces} pcs)</option>)}</select></td><td><input type="number" min="1" value={row.pieces} onChange={e=>updateRow(i,'pieces',e.target.value)} className="border p-1 rounded w-24"/></td><td>{row.pricePerPiece}</td><td>{row.total}</td></tr>))}</tbody></table><button onClick={addRow} className="mt-2 text-blue-600"><i className="fas fa-plus"></i> Add product</button></div>
                <div className="mt-4 border-t pt-4"><p>Total Amount: ৳{totalAmount.toFixed(2)}</p><label>Paid Amount: <input type="number" value={paidAmount} onChange={e=>setPaidAmount(e.target.value)} className="border p-1 rounded"/></label><p>Due: ৳{(totalAmount-paidAmount).toFixed(2)}</p><button onClick={submitInvoice} className="mt-3 bg-green-600 text-white px-5 py-2 rounded-lg">Save Invoice</button></div></div>
            );
        };

        const SaleHistory = ({ sales, customers }) => (<div className="bg-white p-5 rounded-2xl"><h2 className="text-xl font-bold mb-3">📜 Sales History</h2><table className="w-full text-sm"><thead><tr><th>Invoice</th><th>Date</th><th>Customer</th><th>Total</th><th>Paid</th><th>Due</th></tr></thead><tbody>{sales.map(s=>{ const cust = customers.find(c=>c.id===s.customerId); return <tr key={s.id} className="border-b"><td>{s.invoiceNo}</td><td>{s.date}</td><td>{cust?.name}</td><td>{s.totalAmount}</td><td>{s.paidAmount}</td><td>{s.dueAmount}</td></tr>;})}</tbody></table></div>);
        const ReturnHistory = ({ sales }) => (<div className="bg-white p-5 rounded-2xl"><h2>Return History</h2>{sales.length===0? <p>No returns</p> : sales.map(s=> <div key={s.id}>Invoice {s.invoiceNo} Returned amount: {s.returnedAmount}</div>)}</div>);
        const MyStock = ({ products }) => (<div className="bg-white p-5 rounded-2xl overflow-auto"><h2>📦 Stock List</h2><table className="w-full"><thead><tr><th>Name</th><th>Company</th><th>Stock(Pcs)</th><th>Total Sqft</th><th>Sell Price/pc</th></tr></thead><tbody>{products.map(p=> <tr key={p.id}><td>{p.name}</td><td>{p.company}</td><td>{p.stockPieces}</td><td>{(p.stockPieces * p.sqftPerPiece).toFixed(2)}</td><td>{p.sellPricePerPiece}</td></tr>)}</tbody></table></div>);
        const GodownStatus = ({ products }) => (<div><h2>Godown Status</h2><div className="grid grid-cols-2 gap-2">{products.map(p=> <div key={p.id} className="bg-white p-3 rounded shadow">{p.name}: {p.stockPieces} pieces ({(p.stockPieces * p.sqftPerPiece).toFixed(1)} sqft)</div>)}</div></div>);
        const AddCustomer = ({ customers, setCustomers }) => { const [name,setName]=useState(''); const [mobile,setMobile]=useState(''); const [addr,setAddr]=useState(''); const add=()=>{ if(name){ const newCust={ id:'c'+Date.now(), name, mobile, address:addr, dueAmount:0 }; setCustomers([...customers,newCust]); setName(''); setMobile(''); setAddr(''); } }; return (<div className="bg-white p-5 rounded"><h2>➕ Add Customer</h2><input placeholder="Name" value={name} onChange={e=>setName(e.target.value)} className="border p-2 w-full my-1"/><input placeholder="Mobile" value={mobile} onChange={e=>setMobile(e.target.value)} className="border p-2 w-full my-1"/><textarea placeholder="Address" value={addr} onChange={e=>setAddr(e.target.value)} className="border p-2 w-full"/><button onClick={add} className="bg-blue-600 text-white p-2 rounded">Add Customer</button><hr className="my-4"/><h3>Customer List</h3><ul>{customers.map(c=><li key={c.id}>{c.name} - Due: ৳{c.dueAmount}</li>)}</ul></div>); };
        const CustomerReport = ({ customers }) => (<div className="bg-white p-5"><h2>📊 Customer Report</h2><table className="w-full"><thead><tr><th>Name</th><th>Mobile</th><th>Due Amount</th></tr></thead><tbody>{customers.map(c=><tr key={c.id}><td>{c.name}</td><td>{c.mobile}</td><td className="text-red-600">{c.dueAmount}</td></tr>)}</tbody></table></div>);
        const AddCategory = ({ categories, setCategories }) => { const [cat,setCat]=useState(''); const add=()=>{ if(cat) setCategories([...categories,{id:Date.now().toString(), name:cat}]); setCat(''); }; return (<div><input value={cat} onChange={e=>setCat(e.target.value)} placeholder="Category name"/><button onClick={add}>Add</button><ul>{categories.map(c=><li key={c.id}>{c.name}</li>)}</ul></div>); };
        const AddProduct = ({ products, setProducts, categories, companies }) => { const [form,setForm]=useState({name:'', categoryId:'', company:'', sqftPerPiece:1, pcsPerCarton:25, stockPieces:0, purchasePricePerPiece:0, sellPricePerPiece:0 }); const submit=()=>{ setProducts([...products,{...form,id:'prod'+Date.now()}]); alert('Product added'); }; return (<div className="bg-white p-4"><h2>➕ Add Product</h2><input placeholder="Name" onChange={e=>setForm({...form,name:e.target.value})}/><select onChange={e=>setForm({...form,categoryId:e.target.value})}><option>Category</option>{categories.map(c=><option key={c.id}>{c.name}</option>)}</select><input type="number" placeholder="Sqft/Piece" onChange={e=>setForm({...form,sqftPerPiece:parseFloat(e.target.value)})}/><input type="number" placeholder="Stock Pieces" onChange={e=>setForm({...form,stockPieces:parseInt(e.target.value)})}/><input type="number" placeholder="Sell Price/Piece" onChange={e=>setForm({...form,sellPricePerPiece:parseFloat(e.target.value)})}/><button onClick={submit} className="bg-green-600 text-white p-2 rounded">Create Product</button></div>); };
        const PurchaseProduct = ({ products, setProducts, suppliers, setPurchases, purchases }) => { const [prodId,setProdId]=useState(''); const [pieces,setPieces]=useState(0); const [costPerPiece,setCost]=useState(0); const addPurchase=()=>{ const prodIndex=products.findIndex(p=>p.id===prodId); if(prodIndex!==-1){ const updated=[...products]; updated[prodIndex].stockPieces+=pieces; setProducts(updated); const newPurchase={ id:Date.now().toString(), productId:prodId, pieces, totalCost:pieces*costPerPiece, date:new Date().toISOString().slice(0,10), supplier:'Direct' }; setPurchases([newPurchase,...purchases]); alert('Purchase added'); } }; return (<div><h2>Purchase Product</h2><select onChange={e=>setProdId(e.target.value)}><option>Select product</option>{products.map(p=><option key={p.id} value={p.id}>{p.name}</option>)}</select><input type="number" placeholder="Pieces" onChange={e=>setPieces(parseInt(e.target.value))}/><input type="number" placeholder="Cost per piece" onChange={e=>setCost(parseFloat(e.target.value))}/><button onClick={addPurchase} className="bg-indigo-600 text-white p-2">Record Purchase</button></div>); };
        const PurchaseHistory = ({ purchases }) => (<div><h2>Purchase History</h2><ul>{purchases.map(p=><li key={p.id}>ProductID:{p.productId} Pieces:{p.pieces} ৳{p.totalCost} on {p.date}</li>)}</ul></div>);
        const AddSupplier = ({ suppliers, setSuppliers }) => { const [name,setName]=useState(''); const add=()=>{ setSuppliers([...suppliers,{id:'sup'+Date.now(),name,mobile:'',address:'',advance:0,totalDue:0}]); setName(''); }; return (<div><input value={name} onChange={e=>setName(e.target.value)} placeholder="Supplier Name"/><button onClick={add}>Add Supplier</button><ul>{suppliers.map(s=><li>{s.name}</li>)}</ul></div>); };
        const SupplierPayment = ({ suppliers, setSuppliers, cashTransactions }) => (<div><h2>Supplier Payment (Demo)</h2><select><option>Select Supplier</option></select><input placeholder="Amount"/><button>Pay</button></div>);
        const AddCost = ({ costs, setCosts }) => { const [desc,setDesc]=useState(''); const [amt,setAmt]=useState(0); const add=()=>{ setCosts([...costs,{id:Date.now(),description:desc,amount:amt,date:new Date().toISOString().slice(0,10)}]); setDesc(''); setAmt(0); }; return (<div><input placeholder="Description" value={desc} onChange={e=>setDesc(e.target.value)}/><input type="number" placeholder="Amount" value={amt} onChange={e=>setAmt(parseFloat(e.target.value))}/><button onClick={add}>Add Cost</button></div>); };
        const CostReport = ({ costs }) => (<div><h2>Cost Report</h2><p>Total Cost: ৳{costs.reduce((s,c)=>s+c.amount,0)}</p><ul>{costs.map(c=><li>{c.description} - ৳{c.amount} ({c.date})</li>)}</ul></div>);
        const CashInOut = ({ cashTransactions, setCashTransactions }) => { const [type,setType]=useState('in'); const [amount,setAmount]=useState(0); const [reason,setReason]=useState(''); const addTrans=()=>{ setCashTransactions([...cashTransactions,{id:Date.now(),type,amount,reason,date:new Date().toISOString().slice(0,10)}]); }; return (<div><h2>Cash Transaction</h2><select onChange={e=>setType(e.target.value)}><option value="in">Cash In</option><option value="out">Cash Out</option></select><input type="number" placeholder="Amount" onChange={e=>setAmount(parseFloat(e.target.value))}/><input placeholder="Reason" onChange={e=>setReason(e.target.value)}/><button onClick={addTrans}>Record</button><h3>History</h3><ul>{cashTransactions.map(t=><li key={t.id}>{t.type==='in'?'+':'-'} ৳{t.amount} - {t.reason}</li>)}</ul></div>); };
        const ProfitReport = ({ sales, costs, cashTransactions }) => { const totalSales = sales.reduce((s,i)=>s+i.totalAmount,0); const totalCosts = costs.reduce((s,i)=>s+i.amount,0); const grossProfit = totalSales - totalCosts; return (<div className="bg-white p-6 rounded-2xl"><h2>Profit & Daily Account</h2><p>Total Sales Revenue: ৳{totalSales.toFixed(2)}</p><p>Total Expenses (Cost): ৳{totalCosts.toFixed(2)}</p><p className="font-bold text-green-700">Net Profit: ৳{grossProfit.toFixed(2)}</p><hr/><p>Cash Balance Trend: Inflow vs Outflow (simplified)</p></div>); };

        ReactDOM.createRoot(document.getElementById("root")).render(<App />);
    </script>
</body>
</html>

This project was built with [Lovable](https://lovable.dev).

**Live app**: https://tilewise-buddy.lovable.app

## Build with Lovable

Continue developing this project in the [Lovable editor](https://lovable.dev/projects/1791a73c-4983-499e-9f55-d5cf1a83dab1).

- **Ship faster**: describe what you want to build and Lovable handles the code.
- **Stay in sync**: every change made in Lovable is committed straight to this repository.
- **Full ownership**: this code is yours. Push to `main` on GitHub and your changes sync back into Lovable, ready for your next prompt.

## Development

Prefer working locally? You need Node.js and npm — [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating).

```sh
git clone <this-repository-url>
cd <repository-name>
npm i
npm run dev
```
