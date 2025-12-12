// Base de datos de productos
const PRODUCTS = [
    {
        id: 1,
        title: 'Pintalabios Velvet Luxury',
        category: 'LABIOS',
        price: 24.99,
        rating: 4.8,
        popularity: 95,
        img: 'assets/Img/Pintalabios Luxury.png',
        discount: 15
    },
    {
        id: 2,
        title: 'Paleta de Sombras Glam',
        category: 'OJOS',
        price: 34.99,
        rating: 4.9,
        popularity: 98,
        img: 'assets/Img/Paleta de sombras Glam.png',
        discount: 20
    },
    {
        id: 3,
        title: 'Base Flawless HD',
        category: 'ROSTRO',
        price: 29.99,
        rating: 4.7,
        popularity: 92,
        img: 'assets/Img/Base de maquillaje HD.png',
        discount: 10
    },
    {
        id: 4,
        title: 'Set de Brochas Premium',
        category: 'ACCESORIOS',
        price: 49.99,
        rating: 4.6,
        popularity: 85,
        img: 'assets/Img/Set de Brochas Premium.png',
        discount: 25
    },
    {
        id: 5,
        title: 'Mascara de Pestañas Volumen',
        category: 'OJOS',
        price: 19.99,
        rating: 4.5,
        popularity: 88,
        img: 'assets/Img/Mascara de pestañas volumen.png',
        discount: 0
    },
    {
        id: 6,
        title: 'Kit de Skincare Esencial',
        category: 'CUIDADO',
        price: 59.99,
        rating: 4.9,
        popularity: 96,
        img: 'assets/Img/Kit de Skincare Esencial.png',
        discount: 30
    },
    {
        id: 7,
        title: 'Iluminador Rosa Gold',
        category: 'ROSTRO',
        price: 22.99,
        rating: 4.8,
        popularity: 90,
        img: 'assets/Img/Iluminador Rosa Gold.png',
        discount: 12
    },
    {
        id: 8,
        title: 'Colección Luxury Complete',
        category: 'SETS',
        price: 129.99,
        rating: 5,
        popularity: 100,
        img: 'assets/Img/Coleccion luxury complete.png',
        discount: 40
    }
];

// Estado global de la aplicación
let state = {
    view: 'home',
    products: PRODUCTS,
    filter: 'TODO',
    sort: 'popular',
    cart: JSON.parse(localStorage.getItem('cart')) || []
};

// Funciones auxiliares
function formatPrice(price) {
    return '$' + price.toFixed(2);
}

function effectivePrice(product) {
    if (product.discount) {
        return Math.round(product.price * (1 - product.discount / 100) * 100) / 100;
    }
    return product.price;
}

function cartCount() {
    return state.cart.reduce((total, item) => total + item.quantity, 0);
}

function cartTotal() {
    return state.cart.reduce((total, item) => {
        const product = PRODUCTS.find(p => p.id === item.id);
        if (product) {
            return total + (effectivePrice(product) * item.quantity);
        }
        return total;
    }, 0);
}

// Actualizar contador del carrito
function updateCartCount() {
    document.getElementById('cart-count').textContent = cartCount();
}

// Agregar producto al carrito
function agregarCarrito(id, nombre) {
    const existente = state.cart.find(item => item.id === id);
    
    if (existente) {
        existente.quantity += 1;
    } else {
        state.cart.push({
            id: id,
            nombre: nombre,
            quantity: 1
        });
    }
    
    localStorage.setItem('cart', JSON.stringify(state.cart));
    updateCartCount();
    alert(`✓ ${nombre} agregado al carrito`);
}

// Eliminar del carrito
function removeFromCart(id) {
    state.cart = state.cart.filter(item => item.id !== id);
    localStorage.setItem('cart', JSON.stringify(state.cart));
    updateCartCount();
    renderCart();
}

// Actualizar cantidad en el carrito
function updateQuantity(id, quantity) {
    const item = state.cart.find(item => item.id === id);
    if (item) {
        if (quantity <= 0) {
            removeFromCart(id);
        } else {
            item.quantity = quantity;
            localStorage.setItem('cart', JSON.stringify(state.cart));
            renderCart();
        }
    }
}

// Renderizar carrito
function renderCart() {
    const cartContainer = document.getElementById('cart-items');
    if (!cartContainer) return;
    
    if (state.cart.length === 0) {
        cartContainer.innerHTML = '<p class="empty-cart">Tu carrito está vacío. <a href="productos.html">Continúa comprando</a></p>';
        document.getElementById('subtotal').textContent = '$0.00';
        document.getElementById('shipping').textContent = '$0.00';
        document.getElementById('tax').textContent = '$0.00';
        document.getElementById('total').textContent = '$0.00';
        return;
    }
    
    let html = '';
    state.cart.forEach(item => {
        const product = PRODUCTS.find(p => p.id === item.id);
        if (product) {
            const effectPrice = effectivePrice(product);
            const itemTotal = effectPrice * item.quantity;
            
            html += `
                <div class="cart-item">
                    <img src="${product.img}" alt="${product.title}">
                    <div class="item-details">
                        <h4>${product.title}</h4>
                        <p>${formatPrice(effectPrice)} x <input type="number" min="1" value="${item.quantity}" onchange="updateQuantity(${item.id}, this.value)"></p>
                    </div>
                    <div class="item-total">
                        <p>${formatPrice(itemTotal)}</p>
                        <button onclick="removeFromCart(${item.id})" class="remove-btn">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                </div>
            `;
        }
    });
    
    cartContainer.innerHTML = html;
    
    // Actualizar resumen
    const subtotal = cartTotal();
    const shipping = document.querySelector('input[name="shipping"]:checked')?.value === 'express' ? 9.99 : 0;
    const tax = Math.round(subtotal * 0.21 * 100) / 100;
    const total = subtotal + shipping + tax;
    
    document.getElementById('subtotal').textContent = formatPrice(subtotal);
    document.getElementById('shipping').textContent = formatPrice(shipping);
    document.getElementById('tax').textContent = formatPrice(tax);
    document.getElementById('total').textContent = formatPrice(total);
    
    updateCartCount();
}

// Funciones para página de productos
function mostrarProductos(productos) {
    const container = document.getElementById('products-grid');
    if (!container) return;
    
    container.innerHTML = '';
    
    productos.forEach(p => {
        const precioFinal = effectivePrice(p);
        const descuento = p.discount ? `<span class="discount-badge">-${p.discount}%</span>` : '';
        const precioOriginal = p.discount ? `<span class="original-price">${formatPrice(p.price)}</span>` : '';
        
        const html = `
            <div class="product-card">
                ${descuento}
                <div class="product-image">
                    <img src="${p.img}" alt="${p.title}">
                </div>
                <div class="product-info">
                    <h3>${p.title}</h3>
                    <div class="rating">
                        ${'<i class="fas fa-star"></i>'.repeat(Math.floor(p.rating))}
                        <span>(${p.rating})</span>
                    </div>
                    <div class="price">
                        ${precioOriginal}
                        <span class="product-price">${formatPrice(precioFinal)}</span>
                    </div>
                    <button onclick="agregarCarrito(${p.id}, '${p.title}')" class="btn-add-cart">
                        <i class="fas fa-shopping-cart"></i> Agregar
                    </button>
                </div>
            </div>
        `;
        
        container.innerHTML += html;
    });
}

function configurarFiltros() {
    const filterSelect = document.getElementById('filter-category');
    const sortSelect = document.getElementById('sort-by');
    
    if (filterSelect) {
        filterSelect.addEventListener('change', aplicarFiltros);
    }
    
    if (sortSelect) {
        sortSelect.addEventListener('change', aplicarFiltros);
    }
    
    // Setup envío radio buttons si existen
    const shippingRadios = document.querySelectorAll('input[name="shipping"]');
    shippingRadios.forEach(radio => {
        radio.addEventListener('change', renderCart);
    });
}

function aplicarFiltros() {
    const filterSelect = document.getElementById('filter-category');
    const sortSelect = document.getElementById('sort-by');
    
    const filtro = filterSelect ? filterSelect.value : 'TODO';
    const orden = sortSelect ? sortSelect.value : 'popular';
    
    let filtered = PRODUCTS;
    
    // Aplicar filtro de categoría
    if (filtro !== 'TODO') {
        filtered = filtered.filter(p => p.category === filtro);
    }
    
    // Aplicar ordenamiento
    switch(orden) {
        case 'price-low':
            filtered.sort((a, b) => effectivePrice(a) - effectivePrice(b));
            break;
        case 'price-high':
            filtered.sort((a, b) => effectivePrice(b) - effectivePrice(a));
            break;
        case 'rating':
            filtered.sort((a, b) => b.rating - a.rating);
            break;
        case 'popular':
        default:
            filtered.sort((a, b) => b.popularity - a.popularity);
    }
    
    mostrarProductos(filtered);
}

// Renderizar página de inicio
function renderHome() {
    const container = document.getElementById('featured-products');
    if (!container) return;
    
    const destacados = PRODUCTS.slice(0, 3);
    
    let html = '';
    destacados.forEach(p => {
        const precioFinal = effectivePrice(p);
        const descuento = p.discount ? `<span class="discount-badge">-${p.discount}%</span>` : '';
        const precioOriginal = p.discount ? `<span class="original-price">${formatPrice(p.price)}</span>` : '';
        
        html += `
            <div class="product-card">
                ${descuento}
                <div class="product-image">
                    <img src="${p.img}" alt="${p.title}">
                </div>
                <div class="product-info">
                    <h3>${p.title}</h3>
                    <div class="rating">
                        ${'<i class="fas fa-star"></i>'.repeat(Math.floor(p.rating))}
                        <span>(${p.rating})</span>
                    </div>
                    <div class="price">
                        ${precioOriginal}
                        <span class="product-price">${formatPrice(precioFinal)}</span>
                    </div>
                    <button onclick="agregarCarrito(${p.id}, '${p.title}')" class="btn-add-cart">
                        <i class="fas fa-shopping-cart"></i> Agregar
                    </button>
                </div>
            </div>
        `;
    });
    
    container.innerHTML = html;
}

// Inicialización
document.addEventListener('DOMContentLoaded', function() {
    // Actualizar contador del carrito al cargar
    updateCartCount();
    
    // Verificar qué página se está viendo
    const contentArea = document.getElementById('content-area');
    
    if (document.getElementById('featured-products')) {
        renderHome();
    }
    
    if (document.getElementById('products-grid')) {
        mostrarProductos(PRODUCTS);
        configurarFiltros();
    }
    
    if (document.getElementById('cart-items')) {
        renderCart();
        configurarFiltros(); // Para el selector de envío
    }
    
    // Click en carrito
    const cartBtn = document.getElementById('cart-btn');
    if (cartBtn) {
        cartBtn.addEventListener('click', function() {
            window.location.href = 'compras.html';
        });
    }
});
