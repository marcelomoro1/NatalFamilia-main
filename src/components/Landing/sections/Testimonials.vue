<script setup>
const testimonials = [
  {
    name: 'Maria Oliveira',
    quote: 'Minha família ficou encantada com o site! Foi o destaque da nossa ceia de Natal.',
    emoji: '🎄',
    date: 'Natal de 2024'
  },
  {
    name: 'Roberto Santos',
    quote: 'Uma forma moderna e emocionante de desejar boas festas. Recomendo a todos!',
    emoji: '🎅',
    date: 'Natal de 2024'
  },
  {
    name: 'Ana Cláudia',
    quote: 'Muito fácil de fazer e o resultado é profissional. Valeu cada centavo.',
    emoji: '⭐',
    date: 'Natal de 2024'
  },
  {
    name: 'Família Pereira',
    quote: 'Compartilhamos com todos os tios e primos no grupo da família, foi um sucesso!',
    emoji: '🎁',
    date: 'Natal de 2024'
  }
]
</script>

<template>
  <section class="w-full py-16 md:py-24 lg:py-32">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <!-- Header -->
      <div class="text-center space-y-4 mb-16">
        <h2 class="font-poppins text-3xl md:text-4xl lg:text-5xl font-bold text-text-primary">
          Depoimentos de Famílias
        </h2>
        <p class="text-text-light text-lg max-w-2xl mx-auto">
          Veja quem já transformou seu Natal com a gente
        </p>
      </div>

      <!-- Testimonials Grid -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <div
          v-for="(testimonial, index) in testimonials"
          :key="index"
          class="bg-card rounded-2xl p-6 border border-border hover:border-accent/30 transition-all"
        >
          <!-- Quote -->
          <p class="text-text-light text-sm mb-6 italic leading-relaxed">
            "{{ testimonial.quote }}"
          </p>

          <!-- Author -->
          <div class="space-y-3">
            <div class="flex items-center gap-3">
              <div class="w-10 h-10 rounded-full bg-accent/20 flex items-center justify-center text-lg">
                {{ testimonial.emoji }}
              </div>
              <div>
                <p class="font-semibold text-text-primary text-sm">{{ testimonial.name }}</p>
                <p class="text-text-light text-xs">{{ testimonial.date }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped></style>
