<template>
  <form @submit.prevent="submitForm">
    <!-- Add your form fields here -->
    <input type="file" multiple @change="handleFileChange">
    <input type="text" v-model="product.name" placeholder="Product Name">
    <!-- Add other fields as needed -->
    <button type="submit">Submit</button>
  </form>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      product: {
        "name": "Laptop LOQ",
        "description": "A high-performance laptop for gaming and work",
        "price": 1500.00,
        "product_code": "LP123456",
        "quantity": 100,
        "imageLink": "https://bizweb.dktcdn.net/100/362/971/products/s-l1600-d8b1657b-6673-47ca-b7c6-ce3ebe399a27.jpg?v=1696495824197",
        "status": "AVAILABLE",
        "categoryIds": [
          1, 2, null, null
        ],
        "categories": [
          {
            "name": "Hi-tech",
            "description": "Category for all electronic devices including smartphones, laptops, and accessories.",
            "category_code": "ELEC001",
            "imageLink": "https://bizweb.dktcdn.net/100/362/971/products/s-l1600-d8b1657b-6673-47ca-b7c6-ce3ebe399a27.jpg?v=1696495824197",
            "status": "AVAILABLE"
          }
        ]
      },
      files: []
    };
  },
  methods: {
    handleFileChange(event) {
      this.files = Array.from(event.target.files);
    },
    async submitForm() {
      const formData = new FormData();
      formData.append('product', JSON.stringify(this.product));
      this.files.forEach(file => {
        formData.append('files', file);
      });

      try {
        // POST request
        const response = await axios.post('http://localhost:8080/product/img', formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        });
        console.log('Response:', response.data);

        // PUT request (if needed, e.g., for updates)
        // const updateResponse = await axios.put('http://localhost:8080/img', formData, {
        //   headers: {
        //     'Content-Type': 'multipart/form-data'
        //   }
        // });
        // console.log('Update Response:', updateResponse.data);
      } catch (error) {
        console.error('Error:', error);
      }
    }
  }
};
</script>
