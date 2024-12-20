<h1 align="center"> Pagination </h1>

## Step-1: Server Side Route Generation
**File:** `index.js`

```javascript
// Define a route to count the total number of products in the database
app.get('/productsCount', async (req, res) => {
  // Use estimatedDocumentCount to get the total count of documents
  const count = await productCollection.estimatedDocumentCount();
  res.send({ count }); // Send the count as a response
});
```

## Step-2: Create Loader for Item Count in Client Side
**File:** `Main.jsx`

```javascript
{
    path: '/',
    element: <Shop></Shop>,
    // Fetch the total number of products from the server
    loader: () => fetch('http://localhost:5000/productsCount'),
}
```

## Step-3: Pagination Implementation in Component (Shop.jsx)
**File:** `Shop.jsx`

```javascript

// Receive the product count from Main.jsx using useLoaderData
const { count } = useLoaderData();

// Calculate the total number of pages based on the count and items per page
const [itemPerpage, setItemPerpage] = useState(10); // Default items per page
const numberOfPages = Math.ceil(count / itemPerpage);
const pages = [...Array(numberOfPages).keys()]; // Generate page numbers

// State for managing the current page
const [currentPage, setCurrentPage] = useState(0);

// Fetch products when the current page or items per page changes
useEffect(() => {
    // add this line code after url ?page=${currentPage}&size=${itemPerpage}
    fetch(`http://localhost:5000/products?page=${currentPage}&size=${itemPerpage}`)
        .then((res) => res.json())
        .then((data) => setProducts(data));
}, [currentPage, itemPerpage]);

// Handle items per page change
const handleItemPerpage = (e) => {
    const val = parseInt(e.target.value);
    setItemPerpage(val);
    setCurrentPage(0); // Reset to the first page
};

// Navigate to the previous page
const handlePrevious = () => {
    if (currentPage > 0) {
        setCurrentPage(currentPage - 1);
    }
};

// Navigate to the next page
const handleNext = () => {
    if (currentPage < pages.length - 1) {
        setCurrentPage(currentPage + 1);
    }
};

// Render pagination UI
return (
    <div className="pagination">
        {/* Previous Button */}
        <button onClick={handlePrevious} disabled={currentPage === 0}>
            Previous
        </button>

        {/* Page Numbers */}
        {pages.map((page) => (
            <button
                key={page}
                className={currentPage === page ? 'selected' : ''}
                onClick={() => setCurrentPage(page)}
            >
                {page}
            </button>
        ))}

        {/* Next Button */}
        <button onClick={handleNext} disabled={currentPage === pages.length - 1}>
            Next
        </button>

        {/* Items per page dropdown */}
        <select value={itemPerpage} onChange={handleItemPerpage}>
            <option value="5">5</option>
            <option value="10">10</option>
            <option value="20">20</option>
            <option value="30">30</option>
            <option value="50">50</option>
        </select>
    </div>
);
```

## Step-4: 
**File:** `index.jsx`

```javascript

    app.get('/products', async(req, res) => {

        const page = parseInt(req.query.page); 
        const size = parseInt(req.query.size);
        console.log('pagination', page, size);
        
        // Add extra line .skip(page * size).limit(size)
        const result = await productCollection.find().skip(page * size).limit(size).toArray();
        res.send(result);
    })

```

