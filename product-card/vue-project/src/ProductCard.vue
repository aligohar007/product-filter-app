<template>
  <section class="product-Carde">
    <div class="card">
      <h1 class="product-title">{{ title }}</h1>
    </div>
    <slot></slot>
    <div class="filters">
      <div class="input-wrapper">
        <input v-model="textsearch" placeholder="Enter Title" class="inptproduct" />
        <button @click="searchbtn" class="setbtn">Search</button>
      </div>

      <select v-model="selectedCategory" @change="filterByCategory" class="styled-select">
        <option value="">All Categories</option>
        <option v-for="cat in categories" :key="cat" :value="cat">
          {{ cat }}
        </option>
      </select>
    </div>

    <!--  Price Range Slider (HTML Only) -->
    <div class="price-range">
      <label class="range-label">Price Range:</label>
      <div class="range-inputs">
        <input type="range" class="range-slider" min="0" max="1000" v-model="minPrice" />
        <input type="range" class="range-slider" min="0" max="1000" v-model="maxPrice" />
      </div>
      <p class="range-values">Min: ${{ minPrice }} - Max: ${{ maxPrice }}</p>


    </div>
    <div class="price-checker">
      <select class="designe-checke" v-model="priceselect" style="padding: 5px; border-radius: 5px;">
        <option value="" disabled selected>Sort by Price</option>
        <option value="Low price" selected>
          Low price
        </option>
        <option value="High price">
          High price
        </option>
      </select>
    </div>




    <h1>{{ error }}</h1>
    <h1 v-if="loading" class="loading">Loading...</h1>
    <h2>{{ displayerrorsms }}</h2>

    <ul v-if="!loading">
      <li v-for="product in prductdata" :key="product.id" style="list-style: none;">
        <strong>Title: <h2>{{ product.title }}</h2></strong>
        <strong>Description:<p>{{ product.description }}</p></strong>
        <strong>
          <p>Price: ${{ product.price }}</p>
        </strong>
        <img :src="product.image" alt="Product Image" class="setimg" />
        <p>Category: {{ product.category }}</p>
      </li>
    </ul>
  </section>
</template>

<script setup>
import { ref, defineProps } from 'vue';
import { onMounted } from 'vue';
import { watch } from 'vue';
// Define props for the component
defineProps({
  title: {
    type: String,
  },
});
const loading = ref(true);
const prductdata = ref([]);
const selectedCategory = ref('')
const error = ref(null);
const categories = ref([]);
const originalData = ref([]);
const textsearch = ref('');
const displayerrorsms = ref(null);
const minPrice = ref(100);
const maxPrice = ref(800);
const priceselect = ref('');

// Simulate data fetching
const fetchData = (async () => {

  try {
    const response = await fetch('https://fakestoreapi.com/products');
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    const data = await response.json();

    prductdata.value = data;
    // Store the original data for filtering
    originalData.value = data;
    // Extract unique categories from the data
    categories.value = [...new Set(data.map((p) => p.category))]
  } catch (err) {
    console.error('Error fetching data:', err);
    error.value = err.message;
  } finally {
    loading.value = false;
  }
})

const filterByCategory = () => {
  loading.value = true;

  setTimeout(() => {
    if (selectedCategory.value === "") {
      return fetchData();
    }
    else if (originalData.value.length === 0) {
      displayerrorsms.value = "Product Not Found";
    }
    else {
      prductdata.value = originalData.value.filter((p) => {
        return p.category === selectedCategory.value;
      })
    }
    loading.value = false;
  }, 300)

}
const searchbtn = (err) => {
  const searchText = textsearch.value.trim().toLowerCase();

  if (searchText === "") {
    displayerrorsms.value = "Please insert the title and get product title";
    return;
  }
  prductdata.value = originalData.value.filter((p) =>
    p.title.toLowerCase().includes(searchText)
  );
};
// Watch for changes in minPrice and maxPrice to filter products
watch([minPrice, maxPrice], () => {
  if (minPrice.value > maxPrice.value) {
    displayerrorsms.value = "Min value is greater then max value";
  }
  else {
    displayerrorsms.value = null;
    prductdata.value = originalData.value.filter((product) => {
      return product.price >= minPrice.value && product.price <= maxPrice.value;
    })
  }
})
// Watch for changes in priceselect to sort products
watch(priceselect, () => {
  loading.value = true;
  setTimeout(() => {
    if (priceselect.value === "") {
      displayerrorsms.value = "Please select a price option";
      return;
    }
    if (priceselect.value === "Low price") {
      originalData.value.sort((a, b) => a.price - b.price);
    } else if (priceselect.value === "High price") {
      originalData.value.sort((a, b) => b.price - a.price);
    }
    else {
      prductdata.value = originalData.value;
    }
    displayerrorsms.value = null;
    loading.value = false;
  }, 300);
});
// Fetch data when the component is mounted
onMounted(() => {
  setTimeout(fetchData, 1000);
});

</script>

<style scoped>
.product-Carde {
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: hidden;
  padding: 30px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f9f9f9;
  min-height: 100vh;
}

.card {
  text-align: center;
  margin-bottom: 20px;
}

.product-title {
  font-size: 28px;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
}

.filters {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-bottom: 30px;
  width: 100%;
  max-width: 600px;
}

.input-wrapper {
  position: relative;
  width: 100%;
  max-width: 280px;
}

.inptproduct {
  padding: 12px 45px 12px 18px;
  /* space for button on right */
  font-size: 16px;
  border: 2px solid #ccc;
  border-radius: 8px;
  outline: none;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  transition: 0.3s ease;
  background-color: white;
  width: 100%;
  box-sizing: border-box;
}

.setbtn {
  position: absolute;
  top: 50%;
  right: 0px;
  transform: translateY(-50%);
  font-size: 18px;
  cursor: pointer;
  color: white;
  background-color: #007bff;
  padding: 13px;
  border: none;
  border-radius: 5px;
  border-top-left-radius: none;
}


.inptproduct,
.styled-select {
  padding: 12px 18px;
  font-size: 16px;
  border: 2px solid #ccc;
  border-radius: 8px;
  outline: none;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  transition: 0.3s ease;
  background-color: white;
  width: 100%;
  position: relative;
  max-width: 280px;
}

.inptproduct:focus,
.styled-select:focus {
  border-color: #007bff;
  box-shadow: 0 0 8px rgba(0, 123, 255, 0.2);
}

.loading {
  color: #007bff;
  font-size: 1.3em;
  font-weight: bold;
  margin-bottom: 20px;
}

.setimg {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 10px;
  margin-top: 10px;
}

ul {
  width: 100%;
  max-width: 800px;
  padding: 0;
}

li {
  background: white;
  padding: 20px;
  margin-bottom: 15px;
  border: 1px solid #ddd;
  border-radius: 12px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
}

li h2 {
  font-size: 20px;
  margin: 0;
  color: #222;
}

li p {
  margin: 5px 0;
  color: #555;
}

.price-range {
  display: flex;
  flex-direction: column;
  align-items: start;
  max-width: 280px;
  width: 100%;
}

.range-label {
  font-weight: 600;
  margin-bottom: 8px;
  color: #333;
}

.range-inputs {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.range-slider {
  width: 100%;
  -webkit-appearance: none;
  height: 6px;
  background: #ddd;
  border-radius: 4px;
  outline: none;
  transition: background 0.3s;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
  border: none;
}

.range-values {
  font-size: 14px;
  margin-top: 6px;
  color: #555;
}

.price-checker {
  padding: 5px;
}

.designe-checke:focus {
  border-color: #007bff;
  background-color: white;
  outline: none;
}



@media screen and (max-width: 600px) {
  .filters {
    flex-direction: column;
    align-items: center;
  }

  .product-title {
    font-size: 22px;
    font-weight: bold;
  }

  .inptproduct,
  .styled-select {
    max-width: 90%;
  }
}
</style>
