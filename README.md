# Style Shop

Great, let's build upon the detailed plan for the e-commerce application. We will use the outline you provided and go into more details where necessary to ensure a comprehensive understanding of the architecture and implementation.

### E-commerce Website Architecture

#### 1. Project Setup

**Initial Setup:**
```sh
ng new ecommerce-website
cd ecommerce-website
ng add @angular/material
```

**Install Necessary Dependencies:**
```sh
npm install @angular/forms @angular/router @angular/material @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools
```

#### 2. Components

**Components Overview:**

1. **Product Listing and Details**
   - `ProductListComponent`: Displays a list of products.
   - `ProductDetailComponent`: Displays detailed information about a selected product.

2. **Shopping Cart**
   - `CartComponent`: Displays the items in the shopping cart.
   - `CartItemComponent`: Displays individual cart items.

3. **User Authentication**
   - `LoginComponent`: Allows users to log in.
   - `RegisterComponent`: Allows users to register a new account.

4. **Order Management**
   - `OrderHistoryComponent`: Displays past orders.
   - `OrderDetailComponent`: Displays details of a specific order.

**Example: ProductListComponent:**
```typescript
// product-list.component.ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { loadProducts } from '../store/actions/product.actions';
import { selectAllProducts } from '../store/selectors/product.selectors';

@Component({
  selector: 'app-product-list',
  templateUrl: './product-list.component.html',
  styleUrls: ['./product-list.component.css']
})
export class ProductListComponent implements OnInit {
  products$ = this.store.select(selectAllProducts);

  constructor(private store: Store) {}

  ngOnInit(): void {
    this.store.dispatch(loadProducts());
  }
}
```

**Example: ProductListComponent Template:**
```html
<!-- product-list.component.html -->
<div class="product-list">
  <mat-card *ngFor="let product of products$ | async" class="product-card">
    <mat-card-header>
      <mat-card-title>{{ product.name }}</mat-card-title>
      <mat-card-subtitle>{{ product.price | currency }}</mat-card-subtitle>
    </mat-card-header>
    <img mat-card-image [src]="product.imageUrl" alt="{{ product.name }}">
    <mat-card-content>
      <p>{{ product.description }}</p>
    </mat-card-content>
    <mat-card-actions>
      <button mat-button (click)="addToCart(product)">Add to Cart</button>
    </mat-card-actions>
  </mat-card>
</div>
```

#### 3. Services

**ProductService:**
```typescript
// product.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Product } from './product.model';

@Injectable({
  providedIn: 'root'
})
export class ProductService {
  private apiUrl = 'https://api.example.com/products';

  constructor(private http: HttpClient) {}

  getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>(this.apiUrl);
  }

  getProductById(id: number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/${id}`);
  }
}
```

**CartService:**
```typescript
// cart.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { Product } from './product.model';

@Injectable({
  providedIn: 'root'
})
export class CartService {
  private cartItems: Product[] = [];
  private cartItemsSubject = new BehaviorSubject<Product[]>([]);

  getCartItems() {
    return this.cartItemsSubject.asObservable();
  }

  addToCart(product: Product) {
    this.cartItems.push(product);
    this.cartItemsSubject.next(this.cartItems);
  }

  removeFromCart(product: Product) {
    this.cartItems = this.cartItems.filter(item => item.id !== product.id);
    this.cartItemsSubject.next(this.cartItems);
  }
}
```

#### 4. Routing

**Routing Configuration:**
```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ProductListComponent } from './product-list/product-list.component';
import { ProductDetailComponent } from './product-detail/product-detail.component';
import { CartComponent } from './cart/cart.component';
import { LoginComponent } from './login/login.component';
import { RegisterComponent } from './register/register.component';
import { OrderHistoryComponent } from './order-history/order-history.component';
import { AuthGuard } from './auth.guard';

const routes: Routes = [
  { path: '', component: ProductListComponent },
  { path: 'product/:id', component: ProductDetailComponent },
  { path: 'cart', component: CartComponent },
  { path: 'login', component: LoginComponent },
  { path: 'register', component: RegisterComponent },
  { path: 'orders', component: OrderHistoryComponent, canActivate: [AuthGuard] }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

#### 5. Forms

**Reactive Forms Example: Login Component:**
```typescript
// login.component.ts
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { AuthService } from '../auth.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html'
})
export class LoginComponent {
  loginForm: FormGroup;

  constructor(private fb: FormBuilder, private authService: AuthService, private router: Router) {
    this.loginForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', Validators.required]
    });
  }

  onSubmit(): void {
    if (this.loginForm.valid) {
      this.authService.login(this.loginForm.value).subscribe(() => {
        this.router.navigate(['/']);
      });
    }
  }
}
```

#### 6. State Management with NgRx

**Actions, Reducers, and Selectors:**

**product.actions.ts:**
```typescript
import { createAction, props } from '@ngrx/store';
import { Product } from '../product.model';

export const loadProducts = createAction('[Product] Load Products');
export const loadProductsSuccess = createAction('[Product] Load Products Success', props<{ products: Product[] }>());
export const loadProductsFailure = createAction('[Product] Load Products Failure', props<{ error: any }>());
```

**product.reducer.ts:**
```typescript
import { createReducer, on } from '@ngrx/store';
import { loadProducts, loadProductsSuccess, loadProductsFailure } from '../actions/product.actions';
import { Product } from '../product.model';

export interface ProductState {
  products: Product[];
  error: string | null;
}

export const initialState: ProductState = {
  products: [],
  error: null
};

const _productReducer = createReducer(
  initialState,
  on(loadProducts, state => ({ ...state, error: null })),
  on(loadProductsSuccess, (state, { products }) => ({ ...state, products })),
  on(loadProductsFailure, (state, { error }) => ({ ...state, error }))
);

export function productReducer(state, action) {
  return _productReducer(state, action);
}
```

**product.selectors.ts:**
```typescript
import { createSelector } from '@ngrx/store';
import { ProductState } from '../reducers/product.reducer';

export const selectProductState = (state: AppState) => state.products;

export const selectAllProducts = createSelector(
  selectProductState,
  (state: ProductState) => state.products
);
```

#### 7. Authentication

**AuthService:**
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Router } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private apiUrl = 'https://api.example.com/auth';

  constructor(private http: HttpClient, private router: Router) {}

  login(credentials: any): Observable<any> {
    return this.http.post<any>(`${this.apiUrl}/login`, credentials);
  }

  register(user: any): Observable<any> {
    return this.http.post<any>(`${this.apiUrl}/register`, user);
  }

  logout(): void {
    localStorage.removeItem('token');
    this.router.navigate(['/login']);
  }

  isLoggedIn(): boolean {
    return !!localStorage.getItem('token');
  }
}
```

**Auth Guard:**
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {

  constructor(private authService: AuthService, private router: Router) {}

  canActivate(): boolean {
    if (this.authService.isLoggedIn()) {
      return true;
    } else {
      this.router.navigate(['/login']);
      return false;
    }
  }
}
```

#### 8. Order Management

**OrderService:**
Certainly! Let's complete the `OrderService` and provide a more detailed implementation along with the `Order` model.

### `order.service.ts`

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Order } from '../models/order.model';

@Injectable({
  providedIn: 'root'
})
export class OrderService {
  private apiUrl = 'https://api.example.com/orders';

  constructor(private http: HttpClient) {}

  // Place a new order
  placeOrder(order: Order): Observable<Order> {
    return this.http.post<Order>(this.apiUrl, order);
  }

  // Fetch order history for the logged-in user
  getOrderHistory(): Observable<Order[]> {
    return this.http.get<Order[]>(this.apiUrl);
  }

  // Fetch details of a specific order
  getOrderById(orderId: string): Observable<Order> {
    return this.http.get<Order>(`${this.apiUrl}/${orderId}`);
  }
}
```

### `order.model.ts`

```typescript
export interface Order {
  id: string;
  userId: string;
  items: OrderItem[];
  totalAmount: number;
  orderDate: string;
  status: 'Pending' | 'Completed' | 'Cancelled';
}

export interface OrderItem {
  productId: string;
  productName: string;
  quantity: number;
  price: number;
}
```

### Order Components

#### `order-history.component.ts`

```typescript
import { Component, OnInit } from '@angular/core';
import { OrderService } from '../services/order.service';
import { Order } from '../models/order.model';

@Component({
  selector: 'app-order-history',
  templateUrl: './order-history.component.html',
  styleUrls: ['./order-history.component.css']
})
export class OrderHistoryComponent implements OnInit {
  orders: Order[] = [];

  constructor(private orderService: OrderService) {}

  ngOnInit(): void {
    this.orderService.getOrderHistory().subscribe(
      (orders) => this.orders = orders,
      (error) => console.error('Failed to fetch order history', error)
    );
  }
}
```

#### `order-history.component.html`

```html
<div class="order-history">
  <h2>Your Order History</h2>
  <div *ngIf="orders.length === 0">No orders found.</div>
  <div *ngFor="let order of orders">
    <mat-card class="order-card">
      <mat-card-title>Order ID: {{ order.id }}</mat-card-title>
      <mat-card-subtitle>Date: {{ order.orderDate }}</mat-card-subtitle>
      <mat-card-content>
        <div *ngFor="let item of order.items">
          <p>{{ item.productName }} - {{ item.quantity }} x {{ item.price | customCurrency }}</p>
        </div>
        <p><strong>Total: {{ order.totalAmount | customCurrency }}</strong></p>
      </mat-card-content>
      <mat-card-actions>
        <button mat-button [routerLink]="['/order', order.id]">View Details</button>
      </mat-card-actions>
    </mat-card>
  </div>
</div>
```

#### `order-detail.component.ts`

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { OrderService } from '../services/order.service';
import { Order } from '../models/order.model';

@Component({
  selector: 'app-order-detail',
  templateUrl: './order-detail.component.html',
  styleUrls: ['./order-detail.component.css']
})
export class OrderDetailComponent implements OnInit {
  order: Order;

  constructor(private route: ActivatedRoute, private orderService: OrderService) {}

  ngOnInit(): void {
    const orderId = this.route.snapshot.paramMap.get('id');
    if (orderId) {
      this.orderService.getOrderById(orderId).subscribe(
        (order) => this.order = order,
        (error) => console.error('Failed to fetch order details', error)
      );
    }
  }
}
```

#### `order-detail.component.html`

```html
<div class="order-detail" *ngIf="order">
  <h2>Order Details</h2>
  <p><strong>Order ID:</strong> {{ order.id }}</p>
  <p><strong>Order Date:</strong> {{ order.orderDate }}</p>
  <p><strong>Status:</strong> {{ order.status }}</p>
  <h3>Items:</h3>
  <ul>
    <li *ngFor="let item of order.items">
      {{ item.productName }} - {{ item.quantity }} x {{ item.price | customCurrency }}
    </li>
  </ul>
  <p><strong>Total Amount:</strong> {{ order.totalAmount | customCurrency }}</p>
</div>
```

### Routing for Order Components

Update `app-routing.module.ts` to include routes for order history and order detail components.

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ProductListComponent } from './components/product/product-list.component';
import { ProductDetailComponent } from './components/product/product-detail.component';
import { CartComponent } from './components/cart/cart.component';
import { LoginComponent } from './components/user/login.component';
import { RegisterComponent } from './components/user/register.component';
import { OrderHistoryComponent } from './components/order/order-history.component';
import { OrderDetailComponent } from './components/order/order-detail.component';
import { AuthGuard } from './guards/auth.guard';

const routes: Routes = [
  { path: '', component: ProductListComponent },
  { path: 'product/:id', component: ProductDetailComponent },
  { path: 'cart', component: CartComponent },
  { path: 'login', component: LoginComponent },
  { path: 'register', component: RegisterComponent },
  { path: 'orders', component: OrderHistoryComponent, canActivate: [AuthGuard] },
  { path: 'order/:id', component: OrderDetailComponent, canActivate: [AuthGuard] }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Summary

The `OrderService` is now complete, with methods for placing an order, fetching order history, and retrieving details of a specific order. Additionally, we've created components for displaying order history and order details, complete with the necessary HTML templates. The routing has been updated to include paths for these components, ensuring that they are accessible within the application.

This setup provides a comprehensive architecture for handling orders within the e-commerce application, covering both the service layer and the user interface.
## Architecture
Certainly! Here's an organized folder structure for the e-commerce application along with a detailed explanation of each folder's purpose and the placement of the different components, services, and other modules:

### Folder Structure

```
ecommerce-website/
|-- src/
|   |-- app/
|   |   |-- components/
|   |   |   |-- cart/
|   |   |   |   |-- cart.component.ts
|   |   |   |   |-- cart.component.html
|   |   |   |   |-- cart.component.css
|   |   |   |   |-- cart-item/
|   |   |   |   |   |-- cart-item.component.ts
|   |   |   |   |   |-- cart-item.component.html
|   |   |   |   |   |-- cart-item.component.css
|   |   |   |-- product/
|   |   |   |   |-- product-list.component.ts
|   |   |   |   |-- product-list.component.html
|   |   |   |   |-- product-list.component.css
|   |   |   |   |-- product-detail.component.ts
|   |   |   |   |-- product-detail.component.html
|   |   |   |   |-- product-detail.component.css
|   |   |   |-- user/
|   |   |   |   |-- login.component.ts
|   |   |   |   |-- login.component.html
|   |   |   |   |-- login.component.css
|   |   |   |   |-- register.component.ts
|   |   |   |   |-- register.component.html
|   |   |   |   |-- register.component.css
|   |   |   |-- order/
|   |   |       |-- order-history.component.ts
|   |   |       |-- order-history.component.html
|   |   |       |-- order-history.component.css
|   |   |       |-- order-detail.component.ts
|   |   |       |-- order-detail.component.html
|   |   |       |-- order-detail.component.css
|   |   |-- services/
|   |   |   |-- product.service.ts
|   |   |   |-- cart.service.ts
|   |   |   |-- auth.service.ts
|   |   |   |-- order.service.ts
|   |   |-- store/
|   |   |   |-- actions/
|   |   |   |   |-- product.actions.ts
|   |   |   |-- reducers/
|   |   |   |   |-- product.reducer.ts
|   |   |   |   |-- index.ts
|   |   |   |-- selectors/
|   |   |   |   |-- product.selectors.ts
|   |   |   |-- effects/
|   |   |       |-- product.effects.ts
|   |   |-- guards/
|   |   |   |-- auth.guard.ts
|   |   |-- models/
|   |   |   |-- product.model.ts
|   |   |   |-- user.model.ts
|   |   |   |-- order.model.ts
|   |   |-- directives/
|   |   |   |-- highlight.directive.ts
|   |   |-- pipes/
|   |   |   |-- currency.pipe.ts
|   |   |-- app-routing.module.ts
|   |   |-- app.component.ts
|   |   |-- app.component.html
|   |   |-- app.component.css
|   |   |-- app.module.ts
|   |-- assets/
|   |-- environments/
|   |   |-- environment.ts
|   |   |-- environment.prod.ts
|   |-- index.html
|   |-- main.ts
|   |-- styles.scss
|-- angular.json
|-- package.json
|-- tsconfig.json
```

### Explanation of Folder Structure

1. **src/app**: Main folder for the Angular application.
   - **components**: Contains all the reusable and feature-specific components.
     - **cart**: Contains components related to the shopping cart.
       - **cart.component.ts, .html, .css**: Main cart component files.
       - **cart-item**: Subfolder for individual cart item component.
     - **product**: Contains components related to product listing and details.
       - **product-list.component.ts, .html, .css**: Product list component files.
       - **product-detail.component.ts, .html, .css**: Product detail component files.
     - **user**: Contains components for user authentication.
       - **login.component.ts, .html, .css**: Login component files.
       - **register.component.ts, .html, .css**: Register component files.
     - **order**: Contains components for order management.
       - **order-history.component.ts, .html, .css**: Order history component files.
       - **order-detail.component.ts, .html, .css**: Order detail component files.
   
   - **services**: Contains all the services for handling business logic and data operations.
     - **product.service.ts**: Service for managing product data.
     - **cart.service.ts**: Service for managing shopping cart operations.
     - **auth.service.ts**: Service for handling user authentication.
     - **order.service.ts**: Service for managing orders.

   - **store**: Contains all NgRx related files for state management.
     - **actions**: Defines actions for the NgRx store.
       - **product.actions.ts**: Actions for products.
     - **reducers**: Contains reducers for handling state changes.
       - **product.reducer.ts**: Reducer for product state.
       - **index.ts**: Combines all reducers.
     - **selectors**: Selectors for querying state.
       - **product.selectors.ts**: Selectors for product state.
     - **effects**: Defines side effects for the NgRx store.
       - **product.effects.ts**: Effects for product-related actions.

   - **guards**: Contains route guards to protect routes.
     - **auth.guard.ts**: Guard for protecting routes that require authentication.

   - **models**: Defines TypeScript interfaces or classes for data models.
     - **product.model.ts**: Model for products.
     - **user.model.ts**: Model for users.
     - **order.model.ts**: Model for orders.

   - **directives**: Contains custom directives.
     - **highlight.directive.ts**: Example directive for highlighting elements.

   - **pipes**: Contains custom pipes.
     - **currency.pipe.ts**: Example pipe for formatting currency.

   - **app-routing.module.ts**: Defines the routing configuration for the application.

   - **app.component.ts, .html, .css**: Root component of the application.

   - **app.module.ts**: Root module of the application.

2. **assets**: Contains static assets such as images, fonts, etc.

3. **environments**: Contains environment-specific configuration files.
   - **environment.ts**: Development environment configuration.
   - **environment.prod.ts**: Production environment configuration.

4. **index.html**: Main HTML file of the application.

5. **main.ts**: Entry point of the application.

6. **styles.scss**: Global styles for the application.

7. **angular.json**: Angular workspace configuration file.

8. **package.json**: Contains project dependencies and scripts.

9. **tsconfig.json**: TypeScript configuration file.

This structure ensures a clear separation of concerns, making the application scalable and maintainable. Each feature has its own directory, and shared resources like services, models, and state management files are organized into their respective folders.
