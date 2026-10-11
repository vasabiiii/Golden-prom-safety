<template>
  <section class="page-card">
    <h1>Контакты</h1>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque in ipsum id orci porta dapibus. Donec
      sollicitudin molestie malesuada. Nulla porttitor accumsan tincidunt.
    </p>
    <p>
      Praesent sapien massa, convallis a pellentesque nec, egestas non nisi. Mauris blandit aliquet elit, eget
      tincidunt nibh pulvinar a. Quisque velit nisi, pretium ut lacinia in, elementum id enim.
    </p>
  </section>
</template>

<style scoped>
.page-card {
  width: 100%;
  max-width: 100%;
  display: grid;
  grid-template-columns: minmax(0, 1fr);
  gap: 12px;
  box-sizing: border-box;
  background: #ffffff;
  border-radius: 18px;
  padding: 30px;
  box-shadow: 0 16px 40px rgba(15, 23, 42, 0.08);
  border: 1px solid #e2e8f0;
}

h1 {
  margin: 0;
  color: #0f172a;
  font-size: clamp(1.55rem, 4vw, 2rem);
}

p {
  margin: 0;
  color: #475569;
  line-height: 1.7;
}

@media (max-width: 768px) {
  .page-card {
    padding: 22px 16px;
    border-radius: 14px;
  }
}
</style>
