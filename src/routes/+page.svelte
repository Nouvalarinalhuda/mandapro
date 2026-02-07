<script>
    import { onMount, onDestroy } from 'svelte';
    import { fade, fly, slide, scale, crossfade } from 'svelte/transition';
    import { quintOut, elasticOut } from 'svelte/easing';
    import { flip } from 'svelte/animate';

  function getHeroLink(slide) {
  if (slide.cta === 'Profil Madrasah') {
    return 'https://man2kotaprobolinggo.sch.id/category/profil/';
  }

  if (slide.cta === 'Lihat Program') {
    // PRODISTIK / Kerjasama ITS
    return 'https://man2kotaprobolinggo.sch.id/category/prodistik/';
  }

  if (slide.cta === "Info Ma'had") {
    // MA'HAD
    return 'https://mahadnurulhudablog.wordpress.com/';
  }

  // Default fallback
  return '#program';
}


    // --- 1. STATE MANAGEMENT ---
    let y = 0;
    let innerHeight = 0;
    let loading = true;
    let activeTabEkskul = 'semua';
    let mobileMenuOpen = false;
    let currentHeroIndex = 0;
    let showPpdbModal = false;
    
    // Timer untuk Hero Slider
    let heroInterval;
  
    // --- 2. DATA SOURCE (DATABASE SIMULATION) ---
    
    const heroSlides = [
      {
        id: 1,
        image: "/foto/man2.jpg",
        title: "MAN 2 KOTA PROBOLINGGO",
        subtitle: "The Center of Excellence",
        desc: "Mencetak Generasi Islami, Cerdas, dan Berwawasan Global melalui integrasi IMTAQ dan IPTEK.",
        cta: "Profil Madrasah"
        
      },
      {
        id: 2,
        image: "/foto/its.jpeg",
        title: "PRODISTIK - ITS",
        subtitle: "Kerjasama Institut Teknologi Sepuluh Nopember",
        desc: "Satu-satunya Madrasah di Probolinggo dengan program D1 TIK terintegrasi untuk bekal keterampilan digital.",
        cta: "Lihat Program"
      },
      {
        id: 3,
        image: "/foto/mahad.jpg",
        title: "MA'HAD NURUL HUDA",
        subtitle: "Pesantren Kampus",
        desc: "Membangun karakter mandiri dan religius dengan program Tahfidz Al-Qur'an dan kajian Kitab Kuning.",
        cta: "Info Ma'had"
      }
    ];
  
    const stats = [
      { label: "Siswa Aktif", value: 1250, icon: "users" },
      { label: "Guru & Staff", value: 98, icon: "briefcase" },
      { label: "Prestasi 2025", value: 145, icon: "trophy" },
      { label: "Ekstrakurikuler", value: 24, icon: "activity" }
    ];
  // === ANIMATED STATS (ODOMETER EFFECT) ===
let animatedStats = stats.map(stat => ({
  ...stat,
  displayValue: 0
}));

function animateStat(index, duration = 1200) {
  const target = animatedStats[index].value;
  const startTime = performance.now();

  function tick(now) {
    const progress = Math.min((now - startTime) / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3); // easeOutCubic
    animatedStats[index].displayValue = Math.floor(eased * target);

    if (progress < 1) {
      requestAnimationFrame(tick);
    }
  }

  requestAnimationFrame(tick);
}

    const programs = [
      {
        title: "SKS 2 Tahun (4 Semester)",
        desc: "Layanan bagi peserta didik dengan kecerdasan istimewa (CIBI) untuk menyelesaikan masa belajar lebih cepat.",
        icon: "lightning",
        color: "bg-amber-100 text-amber-700",
        features: ["Kelas Eksklusif", "Pemadatan Kurikulum", "Bimbingan Intensif"]
      },
      {
        title: "PRODISTIK ITS",
        desc: "Program Pendidikan Setara D1 Bidang TIK kerjasama dengan ITS Surabaya. Fokus pada Desain Grafis & Pemrograman.",
        icon: "chip",
        color: "bg-blue-100 text-blue-700",
        features: ["Sertifikat ITS", "Lab Komputer Canggih", "Tugas Akhir Karya"]
      },
      {
        title: "Tahfidz Al-Qur'an",
        desc: "Program unggulan menghafal Al-Qur'an dengan target minimal serta pemahaman tafsir dasar.",
        icon: "book",
        color: "bg-emerald-100 text-emerald-700",
        features: ["Target Juz 30, 1, 2", "Murojaah Rutin", "Wisuda Tahfidz"]
      },
      {
        title: "Madrasah Riset",
        desc: "Mewadahi siswa yang gemar meneliti dan berinovasi dalam bidang Sains, Sosial, dan Teknologi Tepat Guna.",
        icon: "microscope",
        color: "bg-purple-100 text-purple-700",
        features: ["Tim MYRES", "KIR", "Robotik"]
      }
    ];
  
    const facilities = [
      { name: "Masjid Nurul Huda", img: "https://images.unsplash.com/photo-1519817650390-64a93db51149?auto=format&fit=crop&w=600&q=80" },
      { name: "Lab. Komputer Multimedia", img: "https://images.unsplash.com/photo-1517694712202-14dd9538aa97?auto=format&fit=crop&w=400&q=80" },
      { name: "Perpustakaan Digital", img: "https://images.unsplash.com/photo-1568667256549-094345857637?auto=format&fit=crop&w=400&q=80" },
      { name: "Gelanggang Olahraga", img: "https://images.unsplash.com/photo-1534438327276-14e5300c3a48?auto=format&fit=crop&w=400&q=80" },
      { name: "Laboratorium MIPA", img: "https://images.unsplash.com/photo-1532094349884-543bc11b234d?auto=format&fit=crop&w=400&q=80" },
      { name: "Asrama Ma'had", img: "https://images.unsplash.com/photo-1555854877-bab0e564b8d5?auto=format&fit=crop&w=400&q=80" }
    ];
  
    const ekskulCategories = [
      { id: 'semua', label: 'Semua' },
      { id: 'krida', label: 'Krida & Lapangan' },
      { id: 'seni', label: 'Seni & Budaya' },
      { id: 'sains', label: 'Sains & Teknologi' },
      { id: 'agama', label: 'Keagamaan' }
    ];
  
    const ekskuls = [
      { name: 'Pramuka', cat: 'krida', desc: 'Membentuk karakter disiplin dan cinta alam.' },
      { name: 'Paskibra', cat: 'krida', desc: 'Pasukan pengibar bendera madrasah.' },
      { name: 'PMR Wira', cat: 'krida', desc: 'Palang Merah Remaja unit kesehatan.' },
      { name: 'Futsal', cat: 'krida', desc: 'Tim Futsal kebanggaan madrasah.' },
      { name: 'Basket', cat: 'krida', desc: 'Tim Basket putra dan putri.' },
      { name: 'Paduan Suara', cat: 'seni', desc: 'Gita Bahana Mandapro.' },
      { name: 'Teater', cat: 'seni', desc: 'Seni peran dan pementasan drama.' },
      { name: 'Hadrah', cat: 'seni', desc: 'Seni musik Islami Al-Banjari.' },
      { name: 'Kaligrafi', cat: 'seni', desc: 'Seni menulis indah huruf Arab.' },
      { name: 'Robotik', cat: 'sains', desc: 'Tim riset dan pengembangan robotika.' },
      { name: 'KIR', cat: 'sains', desc: 'Kelompok Ilmiah Remaja.' },
      { name: 'Jurnalistik', cat: 'sains', desc: 'Tim redaksi majalah dan website.' },
      { name: 'Olimpiade', cat: 'sains', desc: 'Bimbingan khusus OSN/KSM.' },
      { name: 'Qiraah', cat: 'agama', desc: 'Seni membaca Al-Quran.' },
      { name: 'Tahfidz Club', cat: 'agama', desc: 'Komunitas penghafal Quran.' }
    ];
  
    const news = [
      {
        id: 1,
        title: "Gandeng Remas, Ekstra Jurnalistik MAN 2 Kota Probolinggo Sukses Gelar Yoution Vol. 6",
        date: "6 Februari 2026",
        cat: "Prestasi",
        img: "/foto/prestasi.jpg",
        excerpt: "MandaProExist – Semangat kompetisi menyelimuti lingkungan MAN 2 Kota Probolinggo pada Rabu, 4 Februari 2026."
      },
      {
        id: 2,
        title: "MAN 2 Kota Probolinggo Peringati Isra Mi’raj 1447 H, Momentum Penguatan Karakter Siswa",
        date: "23 januari 2026",
        cat: "Keagamaan",
        img: "/foto/agama.jpeg",
        excerpt: "MandaProExist – Madrasah Aliyah Negeri (MAN) 2 Kota Probolinggo menggelar peringatan Isra Mi’raj Nabi Muhammad SAW pada Jumat (23/01)."
      },
      {
        id: 3,
        title: "MAN 2 Kota Probolinggo Gelar “Study Campus 2026”, Jelajahi Kampus Ternama di Malang dan Perkuat Sinergi Lewat MOU",
        date: "30 januari 2026",
        cat: "Akademik",
        img: "/foto/akademik.jpeg",
        excerpt: "MandaProExist-Dalam rangka memperluas wawasan akademik dan mempersiapkan diri menuju jenjang perguruan tinggi"
      }
    ];
  
    const faqs = [
      { q: "Kapan PPDB MAN 2 Kota Probolinggo dibuka?", a: "PPDB Jalur Prestasi biasanya dibuka bulan Februari-Maret, sedangkan Jalur Reguler bulan Mei-Juni. Pantau website resmi kami." },
      { q: "Apa saja syarat masuk program SKS 2 Tahun?", a: "Siswa harus lulus tes psikologi (IQ > 125), memiliki nilai rata-rata rapor MTS/SMP minimal 85, dan lulus tes akademik madrasah." },
      { q: "Berapa biaya masuk ke MAN 2?", a: "MAN 2 adalah madrasah negeri, sehingga tidak ada uang gedung. Biaya hanya untuk seragam dan kegiatan komite yang disepakati bersama wali murid." },
      { q: "Apakah boleh membawa HP ke madrasah?", a: "Siswa diperbolehkan membawa HP untuk keperluan pembelajaran (Digital Learning), namun penggunaannya diatur ketat saat KBM." }
    ];
  
    // --- 3. LOGIC FUNCTIONS ---
  
    onMount(() => {
  // Simulate Loading
  setTimeout(() => {
    loading = false;

    // Jalankan animasi stats setelah loading
    setTimeout(() => {
      animatedStats.forEach((_, i) => {
        animateStat(i, 1200 + i * 200);
      });
    }, 300);

  }, 1500);

  // Hero Slider Loop
  heroInterval = setInterval(() => {
    nextSlide();
  }, 6000);
});

  
    onDestroy(() => {
      if (heroInterval) clearInterval(heroInterval);
    });
  
    function nextSlide() {
      currentHeroIndex = (currentHeroIndex + 1) % heroSlides.length;
    }
  
    function prevSlide() {
      currentHeroIndex = (currentHeroIndex - 1 + heroSlides.length) % heroSlides.length;
    }
  
    function setSlide(index) {
      currentHeroIndex = index;
      // Reset timer
      clearInterval(heroInterval);
      heroInterval = setInterval(nextSlide, 6000);
    }
  
    // Filter Ekskul logic
    $: filteredEkskuls = activeTabEkskul === 'semua' 
      ? ekskuls 
      : ekskuls.filter(e => e.cat === activeTabEkskul);
  
  </script>
  
  <!-- --- 4. WINDOW BINDINGS --- -->
  <svelte:window bind:scrollY={y} bind:innerHeight={innerHeight} />
  
  <svelte:head>
    <title>MAN 2 Kota Probolinggo | Mandapro Exist</title>
    <meta name="description" content="Website Resmi MAN 2 Kota Probolinggo. Madrasah Aliyah Negeri unggulan dengan program Prodistik ITS, SKS, dan Tahfidz." />
    <!-- Preload Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&family=Playfair+Display:ital,wght@0,600;1,600&display=swap" rel="stylesheet">
  </svelte:head>
  
  <!-- --- 5. STYLES & LAYOUT --- -->
  
  {#if loading}
    <!-- LOADING SCREEN -->
    <div class="fixed inset-0 z-[100] bg-emerald-900 flex flex-col items-center justify-center text-white" transition:fade>
      <div class="relative w-24 h-24 mb-4">
        <div class="absolute inset-0 border-4 border-white/20 rounded-full"></div>
        <div class="absolute inset-0 border-4 border-t-emerald-400 border-r-transparent border-b-transparent border-l-transparent rounded-full animate-spin"></div>
        <div class="absolute inset-0 flex items-center justify-center font-bold text-2xl"><img
    src="/foto/kemenag.png"
    alt="Logo Kemenag"
    class="h-12 w-auto object-contain"/>
  </div>
      </div>
      <p class="animate-pulse tracking-[0.2em] text-sm">MEMUAT MANDAPRO...</p>
    </div>
  {:else}
  
    <div class="font-jakarta text-slate-800 bg-slate-50 overflow-x-hidden selection:bg-emerald-500 selection:text-white">
      
      <!-- === NAVBAR === -->
      <nav class={`fixed w-full z-50 transition-all duration-500 border-b ${y > 50 ? 'bg-white/95 backdrop-blur-md shadow-lg border-slate-200 py-3' : 'bg-transparent border-transparent py-6'}`}>
        <div class="container mx-auto px-4 lg:px-8">
          <div class="flex justify-between items-center">
            
            <!-- Logo -->
            <a href="/" class="flex items-center gap-3 group">
  <div class="relative w-10 h-10 overflow-hidden flex items-center justify-center transition-all duration-300">
    <img
      src="/foto/kemenag.png"
      alt="Logo MAN 2 Kota Probolinggo"
      class="h-10 w-auto object-contain"
    />
  </div>

  <div class="flex flex-col">
    <span class={`font-bold text-lg leading-none tracking-tight ${y > 50 ? 'text-slate-800' : 'text-white'}`}>
      MAN 2
    </span>
    <span class={`text-[10px] tracking-[0.2em] font-semibold ${y > 50 ? 'text-emerald-600' : 'text-emerald-300'}`}>
      KOTA PROBOLINGGO
    </span>
  </div>
</a>

  
            <!-- Desktop Menu -->
            <div class="hidden lg:flex items-center gap-1 bg-white/10 backdrop-blur-sm p-1 rounded-full border border-white/10">
              {#each ['Beranda', 'Profil', 'Program', 'Fasilitas', 'Berita', 'Kontak'] as item}
                <a href="#{item.toLowerCase()}" 
                   class={`px-5 py-2 rounded-full text-sm font-medium transition-all duration-300 
                   ${y > 50 ? 'text-slate-600 hover:text-emerald-600 hover:bg-slate-100' : 'text-white/90 hover:text-white hover:bg-white/20'}`}>
                  {item}
                </a>
              {/each}
            </div>
  
            <!-- CTA Button & Mobile Toggle -->
            <div class="flex items-center gap-4">
              <a href="https://man2kotaprobolinggo.sch.id/ppdb-man-2-kota-probolinggo/" 
                target="_blank"
                class="hidden md:flex btn bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-600 hover:to-teal-700 text-white border-none rounded-full px-6 shadow-lg shadow-emerald-500/30 transition-transform hover:scale-105">
                PPDB Online
              </a>

  
              <button class="lg:hidden btn btn-ghost btn-circle" on:click={() => mobileMenuOpen = !mobileMenuOpen}>
                <svg xmlns="http://www.w3.org/2000/svg" class={`h-6 w-6 ${y > 50 ? 'text-slate-800' : 'text-white'}`} fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
                </svg>
              </button>
            </div>
          </div>
        </div>
  
        <!-- Mobile Menu Drawer -->
        {#if mobileMenuOpen}
          <div class="absolute top-full left-0 w-full bg-white shadow-2xl border-t border-slate-100 p-6 flex flex-col gap-4 lg:hidden origin-top" transition:slide>
            {#each ['Beranda', 'Profil', 'Program', 'Fasilitas', 'Berita', 'Kontak'] as item}
              <a href="#{item.toLowerCase()}" class="flex items-center justify-between p-3 hover:bg-emerald-50 rounded-xl text-slate-700 font-medium group" on:click={() => mobileMenuOpen = false}>
                {item}
                <span class="text-emerald-500 opacity-0 group-hover:opacity-100 transition-opacity">➜</span>
              </a>
            {/each}
            <div class="h-px bg-slate-200 my-2"></div>
            <button on:click={() => {showPpdbModal = true; mobileMenuOpen = false;}} class="btn btn-block bg-emerald-600 text-white">Daftar PPDB Sekarang</button>
          </div>
        {/if}
      </nav>
  
      <!-- === HERO SLIDER SECTION === -->
      <section id="beranda" class="relative min-h-screen w-full overflow-hidden bg-slate-900">
        {#each heroSlides as slide, i}
          {#if i === currentHeroIndex}
            <div class="absolute inset-0" transition:crossfade={{duration: 1000}}>
              <!-- Background Image with Zoom Effect -->
              <div class="absolute inset-0 bg-cover bg-center transition-transform duration-[10000ms] ease-linear scale-110" 
                   style="background-image: url('{slide.image}'); transform: scale(1.1);">
              </div>
              
              <!-- Overlays -->
              <div class="absolute inset-0 bg-gradient-to-r from-emerald-950/90 via-slate-900/60 to-transparent"></div>
              <div class="absolute inset-0 bg-gradient-to-t from-slate-900 via-transparent to-transparent"></div>
  
              <!-- Text Content -->
              <div class="absolute inset-0 container mx-auto px-4 flex flex-col justify-center h-full z-20 pt-30 pb-24">
                <div class="max-w-3xl space-y-6">
                  
  
                  <h1 in:fly={{y: 50, duration: 800, delay: 500}} class="text-5xl md:text-7xl lg:text-8xl font-bold text-white leading-tight font-playfair">
                    {slide.title}
                    <span class="block text-2xl md:text-4xl text-emerald-400 font-sans font-light mt-2 italic">{slide.subtitle}</span>
                  </h1>
  
                  <p in:fly={{y: 50, duration: 800, delay: 700}} class="text-lg md:text-xl text-slate-300 max-w-xl leading-relaxed">
                    {slide.desc}
                  </p>
  
                  <div in:fly={{y: 50, duration: 800, delay: 900}} class="flex flex-wrap gap-4 pt-4">
                   <a 
                      href={getHeroLink(slide)}
                      target="_blank"
                      rel="noopener noreferrer"
                      class="btn btn-lg bg-emerald-500 hover:bg-emerald-600 text-white border-none rounded-full px-8 shadow-lg shadow-emerald-500/20 group"
                    >
                      {slide.cta}
                      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 ml-2 group-hover:translate-x-1 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5m0 0l-5 5m5-5H6" />
                      </svg>
                   </a>

                    
                  </div>
                </div>
              </div>
            </div>
          {/if}
        {/each}
  
        <!-- Slider Controls -->
        <div class="absolute bottom-24 right-10 z-30 flex gap-4">
          <button on:click={prevSlide} class="w-12 h-12 rounded-full border border-white/20 text-white flex items-center justify-center hover:bg-emerald-500 hover:border-emerald-500 transition-all">←</button>
          <button on:click={nextSlide} class="w-12 h-12 rounded-full border border-white/20 text-white flex items-center justify-center hover:bg-emerald-500 hover:border-emerald-500 transition-all">→</button>
        </div>
  
        <!-- Slider Indicators -->
        <div class="absolute bottom-10 right-12 z-30 flex gap-2">
          {#each heroSlides as _, i}
            <button on:click={() => setSlide(i)} class={`h-1 transition-all duration-300 rounded-full ${currentHeroIndex === i ? 'w-12 bg-emerald-400' : 'w-4 bg-white/30 hover:bg-white/50'}`}></button>
          {/each}
        </div>
      </section>
  
      <!-- === STATS BANNER === -->
      <div class="relative z-30 - mt-16 container mx-auto px-4">
        <div class="bg-white rounded-3xl shadow-2xl p-8 lg:p-12 grid grid-cols-2 lg:grid-cols-4 gap-8">
          {#each animatedStats as stat}
            <div class="text-center group">
              <div class="w-16 h-16 mx-auto bg-emerald-50 rounded-2xl flex items-center justify-center text-emerald-600 text-3xl mb-4 group-hover:bg-emerald-500 group-hover:text-white transition-all duration-500 group-hover:rotate-6">
                <!-- Simple SVG switch based on icon name -->
                {#if stat.icon === 'users'}
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" /></svg>
                {:else if stat.icon === 'briefcase'}
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 13.255A23.931 23.931 0 0112 15c-3.183 0-6.22-.62-9-1.745M16 6V4a2 2 0 00-2-2h-4a2 2 0 00-2 2v2m4 6h.01M5 20h14a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" /></svg>
                {:else if stat.icon === 'trophy'}
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                {:else}
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z" /></svg>
                {/if}
              </div>
              <div class="text-4xl font-extrabold text-slate-800 mb-1 font-playfair tabular-nums">{stat.displayValue}+ </div>

              <div class="text-sm text-slate-500 font-medium uppercase tracking-wider">{stat.label}</div>
            </div>
          {/each}
        </div>
      </div>
  
      <!-- === PROFILE SECTION (HEADMASTER) === -->
      <section id="profil" class="py-24 relative overflow-hidden">
        <!-- Decor pattern -->
        <div class="absolute top-0 right-0 w-[500px] h-[500px] bg-emerald-50/50 rounded-full blur-3xl -z-10 translate-x-1/2"></div>
  
        <div class="container mx-auto px-4">
          <div class="flex flex-col lg:flex-row items-center gap-16">
            
            <!-- Image Side -->
            <div class="lg:w-1/2 relative group">
              <div class="absolute inset-0 bg-emerald-600 rounded-3xl rotate-3 group-hover:rotate-6 transition-transform duration-500"></div>
              <img src="/foto/kepalasekolah1.jpeg" 
                   alt="Kepala Madrasah" 
                   class="relative rounded-3xl shadow-2xl w-full object-cover h-[500px] z-10 grayscale hover:grayscale-0 transition-all duration-500">
              
              <div class="absolute bottom-8 -right-4 z-20 bg-white p-6 rounded-2xl shadow-xl max-w-xs animate-bounce-slow">
                <div class="flex items-center gap-2 mb-2">
                   <div class="w-3 h-3 bg-emerald-500 rounded-full"></div>
                   <div class="w-3 h-3 bg-yellow-400 rounded-full"></div>
                   <div class="w-3 h-3 bg-red-400 rounded-full"></div>
                </div>
                <p class="text-slate-600 italic text-sm">"Pendidikan adalah senjata paling mematikan di dunia, karena dengan pendidikan Anda dapat mengubah dunia."</p>
              </div>
            </div>
  
            <!-- Text Side -->
            <div class="lg:w-1/2">
              <span class="text-emerald-600 font-bold tracking-wider uppercase mb-2 block">Sambutan Kepala Madrasah</span>
              <h2 class="text-4xl lg:text-5xl font-bold text-slate-800 mb-6 font-playfair">Membangun Peradaban dari <span class="relative whitespace-nowrap"><span class="absolute -bottom-2 left-0 w-full h-3 bg-emerald-200/50 -z-10"></span>Ruang Kelas</span></h2>
              
              <div class="prose prose-lg text-slate-600 mb-8">
                <p>
                  Assalamualaikum Warahmatullahi Wabarakatuh.<br><br>
                  Selamat datang di MAN 2 Kota Probolinggo. Di era disrupsi digital ini, tantangan pendidikan semakin kompleks. Kami tidak hanya berfokus pada transfer pengetahuan, tetapi juga penanaman nilai adab dan akhlak.
                </p>
                <p>
                  Visi kami jelas: <strong>"Terwujudnya Insan Kamil yang Unggul dalam IMTAQ, Terampil dalam IPTEK, dan Peduli Lingkungan."</strong> Melalui sinergi antara kurikulum nasional, kepesantrenan, dan keterampilan vokasi (Prodistik), kami siap mengantarkan putra-putri Anda menuju gerbang kesuksesan dunia dan akhirat.
                </p>
              </div>
  
              <div class="flex items-center gap-6">
                <div>
                  <h4 class="font-bold text-slate-900 text-lg">Abdul Ghofur. M.Pd.I</h4>
                  <p class="text-emerald-600">Kepala Madrasah</p>
                </div>
                
              </div>
            </div>
          </div>
        </div>
      </section>
  
      <!-- === PROGRAMS SECTION === -->
      <section id="program" class="py-24 bg-slate-100">
        <div class="container mx-auto px-4">
          <div class="text-center max-w-3xl mx-auto mb-16">
            <h2 class="text-3xl md:text-5xl font-bold text-slate-800 mb-4 font-playfair">Program Akademik</h2>
            <p class="text-slate-600 text-lg">Kami merancang kurikulum adaptif yang menyesuaikan dengan bakat dan minat peserta didik untuk hasil yang optimal.</p>
          </div>
  
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
            {#each programs as prog}
              <div class="bg-white rounded-2xl p-8 shadow-sm hover:shadow-xl hover:-translate-y-2 transition-all duration-300 group border border-slate-100 relative overflow-hidden">
                <!-- Hover bg effect -->
                <div class={`absolute inset-0 ${prog.color.split(' ')[0]} opacity-0 group-hover:opacity-5 transition-opacity duration-300`}></div>
  
                <div class={`w-14 h-14 ${prog.color} rounded-2xl flex items-center justify-center text-3xl mb-6 shadow-sm`}>
                  <!-- Icons handled dynamically just by generic svg wrapper for demo -->
                  <div class="scale-125">
                     {#if prog.icon === 'lightning'} ⚡ {:else if prog.icon === 'chip'} 💻 {:else if prog.icon === 'book'} 📖 {:else} 🔬 {/if}
                  </div>
                </div>
                
                <h3 class="text-xl font-bold mb-3 text-slate-800 group-hover:text-emerald-600 transition-colors">{prog.title}</h3>
                <p class="text-slate-500 text-sm leading-relaxed mb-6">{prog.desc}</p>
                
                <ul class="space-y-2 mb-6">
                  {#each prog.features as feat}
                    <li class="flex items-center text-xs text-slate-600 font-medium">
                      <svg class="w-4 h-4 text-emerald-500 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path></svg>
                      {feat}
                    </li>
                  {/each}
                </ul>
  
                <button class="w-full py-2 rounded-lg border border-slate-200 text-slate-600 text-sm font-semibold hover:bg-emerald-500 hover:text-white hover:border-emerald-500 transition-all">
                  Detail Program
                </button>
              </div>
            {/each}
          </div>
        </div>
      </section>
  
      <!-- === EKSTRAKURIKULER (Tabs & Grid) === -->
      <section class="py-24 bg-white relative">
        <div class="container mx-auto px-4">
          <div class="flex flex-col md:flex-row justify-between items-end mb-12">
            <div class="max-w-xl">
              <h2 class="text-3xl md:text-5xl font-bold text-slate-800 mb-4 font-playfair">Kembangkan Potensi</h2>
              <p class="text-slate-600">Lebih dari 20 Ekstrakurikuler siap mewadahi bakat non-akademik siswa.</p>
            </div>
            
            <!-- Tabs -->
            <div class="flex flex-wrap gap-2 mt-6 md:mt-0">
              {#each ekskulCategories as cat}
                <button 
                  on:click={() => activeTabEkskul = cat.id}
                  class={`px-4 py-2 rounded-full text-sm font-semibold transition-all ${activeTabEkskul === cat.id ? 'bg-emerald-600 text-white shadow-lg shadow-emerald-500/30' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}`}
                >
                  {cat.label}
                </button>
              {/each}
            </div>
          </div>
  
          <!-- Grid Content with Animation -->
          <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-6 min-h-[400px]">
            {#each filteredEkskuls as item (item.name)}
              <div animate:flip={{duration: 300}} in:scale={{start:0.9, duration:300}} class="bg-slate-50 rounded-xl p-6 border border-slate-100 hover:border-emerald-200 hover:shadow-lg transition-all group cursor-pointer">
                <div class="w-12 h-12 bg-white rounded-full shadow-sm flex items-center justify-center text-2xl mb-4 group-hover:scale-110 transition-transform">
                  {#if item.cat === 'krida'} 🏃 {:else if item.cat === 'seni'} 🎨 {:else if item.cat === 'sains'} 🚀 {:else} 🕌 {/if}
                </div>
                <h4 class="font-bold text-slate-800 mb-2">{item.name}</h4>
                <p class="text-xs text-slate-500 line-clamp-3">{item.desc}</p>
              </div>
            {/each}
          </div>
        </div>
      </section>
  
      <!-- === FACILITIES (Horizontal Scroll) === -->
      <section id="fasilitas" class="py-24 bg-emerald-900 text-white overflow-hidden">
        <div class="container mx-auto px-4 mb-10 flex justify-between items-center">
          <h2 class="text-3xl font-bold font-playfair">Fasilitas Penunjang</h2>
          <div class="hidden md:flex gap-2">
            <span class="text-emerald-400 text-sm">Geser untuk melihat</span>
            <span class="animate-bounce-right">→</span>
          </div>
        </div>
        
        <!-- Scrolling Container -->
        <div class="flex overflow-x-auto pb-12 gap-6 px-4 snap-x snap-mandatory scrollbar-hide">
          {#each facilities as fac, i}
            <div class="min-w-[300px] md:min-w-[400px] h-[300px] relative rounded-2xl overflow-hidden snap-center group cursor-pointer">
              <img src={fac.img} alt={fac.name} class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-700">
              <div class="absolute inset-0 bg-gradient-to-t from-black/80 via-transparent to-transparent"></div>
              <div class="absolute bottom-0 left-0 p-6">
                <div class="text-emerald-400 text-xs font-bold uppercase tracking-wider mb-1">Fasilitas 0{i+1}</div>
                <h3 class="text-2xl font-bold">{fac.name}</h3>
              </div>
            </div>
          {/each}
          <!-- Padding end -->
          <div class="min-w-[50px]"></div>
        </div>
      </section>
  
      <!-- === NEWS SECTION === -->
      <section id="berita" class="py-24 bg-slate-50">
        <div class="container mx-auto px-4">
          <div class="text-center mb-16">
             <span class="text-emerald-600 font-bold tracking-wider text-sm">UPDATES</span>
             <h2 class="text-4xl font-bold text-slate-800 font-playfair mt-2">Kabar Mandapro</h2>
          </div>
  
          <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
            {#each news as n}
              <article class="bg-white rounded-2xl overflow-hidden shadow-sm hover:shadow-2xl transition-all duration-300 group flex flex-col h-full">
                <div class="relative h-56 overflow-hidden">
                  <img src={n.img} alt={n.title} class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500">
                  <div class="absolute top-4 left-4 bg-white/90 backdrop-blur-md px-3 py-1 rounded-full text-xs font-bold text-emerald-800 shadow-sm">
                    {n.cat}
                  </div>
                </div>
                <div class="p-6 flex-1 flex flex-col">
                  <div class="flex items-center gap-2 text-xs text-slate-400 mb-3">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                    {n.date}
                  </div>
                  <h3 class="text-xl font-bold text-slate-800 mb-3 line-clamp-2 group-hover:text-emerald-600 transition-colors">
                    <a href="#read">{n.title}</a>
                  </h3>
                  <p class="text-slate-500 text-sm line-clamp-3 mb-4 flex-1">{n.excerpt}</p>
                  <a href="https://man2kotaprobolinggo.sch.id/category/event/" class="inline-flex items-center text-emerald-600 font-semibold text-sm hover:underline">
                    Baca Selengkapnya <span class="ml-1 transition-transform group-hover:translate-x-1">→</span>
                  </a>
                </div>
              </article>
            {/each}
          </div>
          
          <div class="text-center mt-12"><a href="https://man2kotaprobolinggo.sch.id/category/event/">

            <button class="btn btn-outline btn-wide border-slate-300 text-slate-600 hover:bg-emerald-600 hover:text-white hover:border-emerald-600">Lihat Arsip Berita</button>
          </a>
          </div>
        </div>
      </section>
  
      <!-- === FAQ & MAP SECTION === -->
      <section id="kontak" class="py-24 bg-white">
        <div class="container mx-auto px-4">
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-16">
            
            <!-- Left: FAQ -->
            <div>
              <h2 class="text-3xl font-bold mb-8 font-playfair">Pertanyaan Umum</h2>
              <div class="space-y-4">
                {#each faqs as faq}
                  <div tabindex="0" class="collapse collapse-plus border border-slate-200 bg-white rounded-xl focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 transition-all">
                    <div class="collapse-title text-lg font-medium text-slate-800">
                      {faq.q}
                    </div>
                    <div class="collapse-content"> 
                      <p class="text-slate-600 leading-relaxed">{faq.a}</p>
                    </div>
                  </div>
                {/each}
              </div>
  
              <div class="mt-10 bg-emerald-50 p-8 rounded-2xl border border-emerald-100">
                <h4 class="font-bold text-emerald-900 mb-2">Butuh Bantuan Lebih Lanjut?</h4>
                <p class="text-emerald-700 text-sm mb-4">Tim administrasi kami siap membantu pertanyaan Anda via WhatsApp.</p>
                <button class="btn bg-emerald-600 text-white hover:bg-emerald-700 w-full border-none">Chat WhatsApp Admin</button>
              </div>
            </div>
  
            <!-- Right: Map & Form -->
            <div>
              <h2 class="text-3xl font-bold mb-8 font-playfair">Lokasi Kami</h2>
              <div class="w-full h-64 rounded-2xl overflow-hidden mb-8 shadow-inner border border-slate-200">
                <iframe
                  title="Lokasi MAN 2 Kota Probolinggo"
                  src="https://www.google.com/maps?q=MAN+2+Kota+Probolinggo&output=embed"
                  class="w-full h-full border-0"
                  loading="lazy"
                  referrerpolicy="no-referrer-when-downgrade">
                </iframe>
              </div>



  
              <form class="space-y-4">
                 <div class="grid grid-cols-2 gap-4">
                   <div class="form-control">
                     <label class="label"><span class="label-text">Nama Lengkap</span></label>
                     <input type="text" placeholder="Nama Anda" class="input input-bordered w-full focus:input-success bg-slate-50" />
                   </div>
                   <div class="form-control">
                     <label class="label"><span class="label-text">Email / No. HP</span></label>
                     <input type="text" placeholder="Kontak" class="input input-bordered w-full focus:input-success bg-slate-50" />
                   </div>
                 </div>
                 <div class="form-control">
                   <label class="label"><span class="label-text">Pesan</span></label>
                   <textarea class="textarea textarea-bordered h-24 focus:textarea-success bg-slate-50" placeholder="Tulis pesan Anda disini..."></textarea>
                 </div>
                 <button class="btn btn-primary w-full bg-slate-800 hover:bg-slate-900 border-none">Kirim Pesan</button>
              </form>
            </div>
          </div>
        </div>
      </section>
  
      <!-- === FOOTER === -->
      <footer class="bg-slate-900 text-slate-300 pt-20 pb-10 border-t-4 border-emerald-500">
        <div class="container mx-auto px-4">
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-12 mb-16">
            
            <!-- Brand Column -->
            <div class="space-y-6">
              <div class="flex items-center gap-3 text-white">
                <div class="w-10 h-10 rounded flex items-center justify-center font-bold text-xl"><img
                  src="/foto/kemenag.png"
                  alt="Logo Kemenag"
                  class="h-8 w-auto object-contain"/>
                </div>
                <span class="font-bold text-xl tracking-wider">MAN 2 PROBOLINGGO</span>
              </div>
              <p class="text-sm leading-relaxed text-slate-400">
                Madrasah Aliyah Negeri 2 Kota Probolinggo adalah lembaga pendidikan yang berkomitmen mencetak generasi unggul, berkarakter Islami, dan siap bersaing di era global.
              </p>
              <div class="flex gap-4">
              
                    <!-- Facebook -->
                    <a href="https://www.facebook.com/profile.php?id=100068654674132&ref=py_c&eid=ARC183obnDdPYcvz1IerGMpItC193irt_AtsZ5WiuW4fNngg07Q8MdpbNMUmzlz_H-3oXBSF6VMzmz__#" target="_blank"
                      class="w-10 h-10 rounded-full bg-slate-800 flex items-center justify-center hover:bg-emerald-600 hover:text-white transition-all">
                      <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M22 12a10 10 0 10-11.5 9.9v-7h-2v-3h2V9.5c0-2 1.2-3.1 3-3.1.9 0 1.8.1 1.8.1v2h-1c-1 0-1.3.6-1.3 1.2V12h2.3l-.4 3h-1.9v7A10 10 0 0022 12z"/>
                      </svg>
                    </a>

                    <!-- Instagram -->
                    <a href="https://www.instagram.com/man2kotaprobolinggo?utm_medium=copy_link" target="_blank"
                      class="w-10 h-10 rounded-full bg-slate-800 flex items-center justify-center hover:bg-emerald-600 hover:text-white transition-all">
                      <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M7 2C4.2 2 2 4.2 2 7v10c0 2.8 2.2 5 5 5h10c2.8 0 5-2.2 5-5V7c0-2.8-2.2-5-5-5H7zm10 2c1.7 0 3 1.3 3 3v10c0 1.7-1.3 3-3 3H7c-1.7 0-3-1.3-3-3V7c0-1.7 1.3-3 3-3h10zm-5 3a5 5 0 100 10 5 5 0 000-10zm0 2a3 3 0 110 6 3 3 0 010-6zm4.8-2.3a1.2 1.2 0 100 2.4 1.2 1.2 0 000-2.4z"/>
                      </svg>
                    </a>

                    <!-- YouTube -->
                    <a href="https://www.youtube.com/c/MAPlusMANDAPROEXISTOfficial?app=desktop" target="_blank"
                      class="w-10 h-10 rounded-full bg-slate-800 flex items-center justify-center hover:bg-emerald-600 hover:text-white transition-all">
                      <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M23.5 6.2s-.2-1.7-.9-2.4c-.8-.9-1.7-.9-2.1-1C17.4 2.5 12 2.5 12 2.5h0s-5.4 0-8.5.3c-.4.1-1.3.1-2.1 1C.7 4.5.5 6.2.5 6.2S.2 8.1.2 10v1.9c0 1.9.3 3.8.3 3.8s.2 1.7.9 2.4c.8.9 1.9.9 2.4 1 1.7.2 7.2.3 7.2.3s5.4 0 8.5-.3c.4-.1 1.3-.1 2.1-1 .7-.7.9-2.4.9-2.4s.3-1.9.3-3.8V10c0-1.9-.3-3.8-.3-3.8zM9.8 14.6V8.4l5.6 3.1-5.6 3.1z"/>
                      </svg>
                    </a>

                    <!-- Twitter / X -->
                    <a href="https://www.addtoany.com/add_to/twitter?linkurl=https%3A%2F%2Fman2kotaprobolinggo.sch.id%2F&linkname=MAN%202%20KOTA%20PROBOLINGGO&linknote=" target="_blank"
                      class="w-10 h-10 rounded-full bg-slate-800 flex items-center justify-center hover:bg-emerald-600 hover:text-white transition-all">
                      <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M18.2 2H21l-6.6 7.5L22 22h-6.4l-5-6.2L5 22H2l7.1-8L2 2h6.6l4.6 5.7L18.2 2z"/>
                      </svg>
                    </a>
                  
                </div>
            </div>
  
            <!-- Quick Links -->
            <div>
              <h4 class="text-white font-bold text-lg mb-6">Tautan Akademik</h4>
              <ul class="space-y-3 text-sm">
                <li><a href="#" class="hover:text-emerald-400 transition-colors flex items-center gap-2"><span>›</span> Kalender Akademik</a></li>
                <li><a href="#" class="hover:text-emerald-400 transition-colors flex items-center gap-2"><span>›</span> E-Learning</a></li>
                <li><a href="#" class="hover:text-emerald-400 transition-colors flex items-center gap-2"><span>›</span> Rapor Digital (RDM)</a></li>
                <li><a href="#" class="hover:text-emerald-400 transition-colors flex items-center gap-2"><span>›</span> Perpustakaan</a></li>
                <li><a href="#" class="hover:text-emerald-400 transition-colors flex items-center gap-2"><span>›</span> Cek Kelulusan</a></li>
              </ul>
            </div>
  
            <!-- Contact Info -->
            <div>
               <h4 class="text-white font-bold text-lg mb-6">Hubungi Kami</h4>
               <ul class="space-y-4 text-sm">
                 <li class="flex items-start gap-3">
                   <span class="text-emerald-500 mt-1">📍</span>
                   <span>Jalan Soekarno - Hatta No.255,<br>Curahgrinting, Kec. Kanigaran,<br>Kota Probolinggo, Jawa Timur 67212</span>
                 </li>
                 <li class="flex items-center gap-3">
                   <span class="text-emerald-500">📞</span>
                   <span>(0335) 421842</span>
                 </li>
                 <li class="flex items-center gap-3">
                   <span class="text-emerald-500">✉️</span>
                   <span>admin@man2kotaprobolinggo.sch.id</span>
                 </li>
               </ul>
            </div>
  
            <!-- Newsletter -->
            <div>
              <h4 class="text-white font-bold text-lg mb-6">Buletin Madrasah</h4>
              <p class="text-sm text-slate-400 mb-4">Dapatkan info terbaru seputar kegiatan dan prestasi siswa.</p>
              <div class="join w-full">
                <input class="input input-bordered join-item w-full bg-slate-800 border-slate-700 text-white focus:bg-slate-700" placeholder="Email Address"/>
                <button class="btn join-item bg-emerald-600 border-emerald-600 text-white hover:bg-emerald-700">Sub</button>
              </div>
            </div>
          </div>
  
          <div class="border-t border-slate-800 pt-8 flex flex-col md:flex-row justify-between items-center text-xs text-slate-500">
            <p>&copy; 2026 MAN 2 Kota Probolinggo. All rights reserved.</p>
            <div class="flex gap-6 mt-4 md:mt-0">
               <a href="#" class="hover:text-white">Privacy Policy</a>
               <a href="#" class="hover:text-white">Terms of Use</a>
               <a href="#" class="hover:text-white">Sitemap</a>
            </div>
          </div>
        </div>
      </footer>
  
      <!-- === FLOATING ACTION BUTTON (WA) === -->
      <a href="https://wa.me/628123456789" class="fixed bottom-6 right-6 z-40 btn btn-circle btn-lg bg-green-500 hover:bg-green-600 border-none text-white shadow-2xl animate-bounce-slow">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="currentColor" viewBox="0 0 24 24">
          <path d="M.057 24l1.687-6.163c-1.041-1.804-1.588-3.849-1.587-5.946.003-6.556 5.338-11.891 11.893-11.891 3.181.001 6.167 1.24 8.413 3.488 2.245 2.248 3.481 5.236 3.48 8.414-.003 6.557-5.338 11.892-11.893 11.892-1.99-.001-3.951-.5-5.688-1.448l-6.305 1.654zm6.597-3.807c1.676.995 3.276 1.591 5.392 1.592 5.448 0 9.886-4.434 9.889-9.885.002-5.462-4.415-9.89-9.881-9.892-5.452 0-9.887 4.434-9.889 9.884-.001 2.225.651 3.891 1.746 5.634l-.999 3.648 3.742-.981zm11.387-5.464c-.074-.124-.272-.198-.57-.347-.297-.149-1.758-.868-2.031-.967-.272-.099-.47-.149-.669.149-.198.297-.768.967-.941 1.165-.173.198-.347.223-.644.074-.297-.149-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.297-.347.446-.521.151-.172.2-.296.3-.495.099-.198.05-.372-.025-.521-.075-.148-.669-1.611-.916-2.206-.242-.579-.487-.506-.669-.51-.173-.004-.371-.004-.569-.004-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.095 3.2 5.076 4.487.709.306 1.263.489 1.694.626.712.226 1.36.194 1.872.118.571-.085 1.758-.719 2.006-1.413.248-.695.248-1.29.173-1.414z"/>
        </svg>
      </a>
  
   
  
    </div>
  {/if}
  
  <style>

    .tabular-nums {
  font-variant-numeric: tabular-nums;
  letter-spacing: 0.04em;
}

    /* Custom Fonts Helper */
    .font-playfair {
      font-family: 'Playfair Display', serif;
    }
    .font-jakarta {
      font-family: 'Plus Jakarta Sans', sans-serif;
    }
    
    /* Hide Scrollbar but keep functionality */
    .scrollbar-hide::-webkit-scrollbar {
        display: none;
    }
    .scrollbar-hide {
        -ms-overflow-style: none;
        scrollbar-width: none;
    }
  
    /* Animations */
    @keyframes bounce-right {
      0%, 100% { transform: translateX(0); }
      50% { transform: translateX(5px); }
    }
    .animate-bounce-right {
      animation: bounce-right 1s infinite;
    }
  
    @keyframes bounce-slow {
      0%, 100% { transform: translateY(-5%); }
      50% { transform: translateY(5%); }
    }
    .animate-bounce-slow {
      animation: bounce-slow 3s infinite ease-in-out;
    }
  </style>