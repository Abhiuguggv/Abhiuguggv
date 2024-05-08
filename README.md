import React, { useState, useEffect } from 'react';
import { BrowserRouter as Router, Route, Switch, Link } from 'react-router-dom';
import axios from 'axios';
import { makeStyles } from '@material-ui/core/styles';
import { Grid, Card, CardContent, Typography, Button, Slider, Checkbox, FormControlLabel } from '@material-ui/core';

const useStyles = makeStyles((theme) => ({
  card: {
    height: '100%',
    display: 'flex',
    flexDirection: 'column',
  },
  cardContent: {
    flexGrow: 1,
  },
  link: {
    textDecoration: 'none',
    color: 'inherit',
  },
}));

const ProductList = () => {
  const classes = useStyles();
  const [products, setProducts] = useState([]);
  const [categories, setCategories] = useState([]);
  const [companies, setCompanies] = useState([]);
  const [filters, setFilters] = useState({
    category: '',
    company: '',
    rating: [0, 5],
    price: [0, 1000],
    available: false,
  });

  useEffect(() => {
    const fetchProducts = async () => {
      try {
        const response = await axios.get('/api/products');
        setProducts(response.data.products);
        setCategories([...new Set(response.data.products.map((product) => product.category))]);
        setCompanies([...new Set(response.data.products.map((product) => product.company))]);
      } catch (error) {
        console.error('Error fetching products:', error);
      }
    };
    fetchProducts();
  }, []);

  const handleFilterChange = (filter, value) => {
    setFilters((prevFilters) => ({
      ...prevFilters,
      [filter]: value,
    }));
  };

  const filteredProducts = products.filter((product) => {
    return (
      (filters.category === '' || product.category === filters.category) &&
      (filters.company === '' || product.company === filters.company) &&
      product.rating >= filters.rating[0] &&
      product.rating <= filters.rating[1] &&
      product.price >= filters.price[0] &&
      product.price <= filters.price[1] &&
      (!filters.available || product.available)
    );
  });

  return (
    <div>
      <h1>Top N Products</h1>
      <Grid container spacing={3}>
        <Grid item xs={12} sm={3}>
          <h3>Filters</h3>
          <div>
            <Typography>Category</Typography>
            <select value={filters.category} onChange={(e) => handleFilterChange('category', e.target.value)}>
              <option value="">All</option>
              {categories.map((category) => (
                <option key={category} value={category}>
                  {category}
                </option>
              ))}
            </select>
          </div>
          <div>
            <Typography>Company</Typography>
            <select value={filters.company} onChange={(e) => handleFilterChange('company', e.target.value)}>
              <option value="">All</option>
              {companies.map((company) => (
                <option key={company} value={company}>
                  {company}
                </option>
              ))}
            </select>
          </div>
          <div>
            <Typography>Rating</Typography>
            <Slider
              value={filters.rating}
              onChange={(_, value) => handleFilterChange('rating', value)}
              min={0}
              max={5}
              step={0.5}
              marks
            />
          </div>
          <div>
            <Typography>Price</Typography>
            <Slider
              value={filters.price}
              onChange={(_, value) => handleFilterChange('price', value)}
              min={0}
              max={1000}
              step={10}
              marks
            />
          </div>
          <div>
            <FormControlLabel
              control={
                <Checkbox
                  checked={filters.available}
                  onChange={(e) => handleFilterChange('available', e.target.checked)}
                />
              }
              label="Available"
            />
</div>
        </Grid>
        <Grid item xs={12} sm={9}>
          <Grid container spacing={3}>
            {filteredProducts.map((product) => (
              <Grid item key={product.id} xs={12} sm={6} md={4}>
                <Card className={classes.card}>
                  <Link to={`/product/${product.id}`} className={classes.link}>
                    <CardContent className={classes.cardContent}>
                      <Typography gutterBottom variant="h5" component="h2">
                        {product.name}
                      </Typography>
                      <Typography variant="body2" color="textSecondary" component="p">
                        Company: {product.company}
                      </Typography>
                      <Typography variant="body2" color="textSecondary" component="p">
                        Category: {product.category}
                      </Typography>
                      <Typography variant="body2" color="textSecondary" component="p">
                        Price: ${product.price}
                      </Typography>
                      <Typography variant="body2" color="textSecondary" component="p">
                        Rating: {product.rating}
                      </Typography>
                      <Typography variant="body2" color="textSecondary" component="p">
                        Discount: {product.discount}%
                      </Typography>
                      <Typography variant="body2" color="textSecondary" component="p">
                        Available: {product.available ? 'Yes' : 'No'}
                      </Typography>
                    </CardContent>
                  </Link>
                </Card>
              </Grid>
            ))}
          </Grid>
        </Grid>
      </Grid>
    </div>
  );
};

const ProductDetails = ({ match }) => {
  const classes = useStyles();
  const [product, setProduct] = useState(null);

  useEffect(() => {
    const fetchProduct = async () => {
      try {
        const response = await axios.get(`/api/products/${match.params.id}`);
        setProduct(response.data);
      } catch (error) {
        console.error('Error fetching product:', error);
      }
    };
    fetchProduct();
  }, [match.params.id]);

  if (!product) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <Grid container spacing={3}>
        <Grid item xs={12} sm={6}>
          <img src={`/images/${product.id}.jpg`} alt={product.name} style={{ maxWidth: '100%' }} />
        </Grid>
        <Grid item xs={12} sm={6}>
          <Typography variant="h6">Company: {product.company}</Typography>
          <Typography variant="h6">Category: {product.category}</Typography>
          <Typography variant="h6">Price: ${product.price}</Typography>
          <Typography variant="h6">Rating: {product.rating}</Typography>
          <Typography variant="h6">Discount: {product.discount}%</Typography>
          <Typography variant="h6">Available: {product.available ? 'Yes' : 'No'}</Typography>
          <Button variant="contained" color="primary">
            Add to Cart
          </Button>
        </Grid>
      </Grid>
    </div>
  );
};

const App = () => {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Product List</Link>
            </li>
          </ul>
        </nav>

        <Switch>
          <Route path="/product/:id">
            <ProductDetails />
          </Route>
          <Route path="/">
            <ProductList />
          </Route>
        </Switch>
      </div>
    </Router>
  );
};

export default App;
