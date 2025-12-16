<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fedochini Periodic Table</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Configure Tailwind to use dark mode and 'Inter' font -->
    <script>
        tailwind.config = {
            darkMode: 'class', // Enable dark mode
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        // Custom colors for element categories
                        'alkali-metal': '#FF6B6B',     // Bright Red
                        'alkaline-earth': '#FFA500',   // Orange
                        'transition-metal': '#FFD700', // Gold
                        'post-transition': '#3CB371',  // Medium Sea Green
                        'metalloid': '#6A5ACD',        // Slate Blue
                        'nonmetal': '#4682B4',         // Steel Blue
                        'halogen': '#00CED1',          // Dark Turquoise
                        'noble-gas': '#DA70D6',        // Orchid
                        'lanthanide': '#90EE90',       // Light Green
                        'actinide': '#BA55D3',         // Medium Orchid
                        'unknown': '#5A5A5A',          // Dark Gray for unknown
                    }
                }
            }
        }
    </script>
    <style>
        /* Base styles for the periodic table grid */
        .periodic-table {
            display: grid;
            grid-template-columns: repeat(18, minmax(20px, 1fr)); /* 18 columns */
            gap: 4px;
            padding: 20px;
            width: 100%;
            max-width: 1400px;
            margin: auto;
        }

        /* Styling for the element cell */
        .element-cell {
            /* OLED Styling: Pure Black Background */
            background-color: #000000; 
            border: 2px solid; /* Border color is defined by category class */
            
            /* Cell structure and transitions */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            aspect-ratio: 1 / 1; 
            cursor: pointer;
            transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
            border-radius: 6px;
            font-size: 0.75rem; 
            line-height: 1;
            padding: 2px;
            font-weight: 500;
            position: relative; 
            z-index: 1; /* Default Z-index */
        }

        /* Hover effect: Pop up and glow shadow */
        .element-cell:hover {
            transform: scale(1.08); /* Pop up effect */
            z-index: 10;
            /* Box shadow to create a strong, colored edge glow effect on hover */
            box-shadow: 0 0 15px 3px var(--element-glow-color); 
        }

        /* Glowing Effect */
        .element-glow {
            /* Create a central background glow (subtle, less distracting than the box-shadow) */
            content: '';
            position: absolute;
            top: 25%;
            left: 25%;
            width: 50%;
            height: 50%;
            opacity: 0;
            transition: opacity 0.3s ease-in-out;
            pointer-events: none; 
            border-radius: 50%;
            filter: blur(10px);
            z-index: 0;
        }

        /* Activate central glow on hover */
        .element-cell:hover .element-glow {
            opacity: 0.8; /* Visible glow */
            animation: pulse-glow 2s infinite alternate;
        }

        @keyframes pulse-glow {
            from { opacity: 0.5; }
            to { opacity: 0.9; }
        }

        /* Modal specific styling (Pure Black BG) */
        .modal-bg {
            background-color: rgba(0, 0, 0, 0.95); /* Pure black with high opacity */
        }
        .modal-content-card {
            background-color: #0d0d0d; /* Slightly off-black for separation */
        }
        
        /* Inner cell content styling */
        .element-symbol { font-size: 1.5rem; font-weight: 900; }
        .element-name { font-size: 0.5rem; font-weight: 300; text-transform: uppercase; letter-spacing: 0.5px; }
        .element-number { position: absolute; top: 2px; left: 4px; font-size: 0.6rem; font-weight: 700; }
        .element-mass { position: absolute; bottom: 2px; right: 4px; font-size: 0.5rem; font-weight: 400; }

        .modal-open { overflow: hidden; }
    </style>
</head>
<body class="bg-black text-gray-100 font-sans p-4 antialiased">

    <!-- Header -->
    <header class="text-center mb-8">
        <h1 class="text-4xl font-extrabold text-white">The OLED Glowing Periodic Table</h1>
        <p class="text-gray-400 mt-2">Pure black mode with colored, glowing borders.</p>
    </header>

    <!-- Periodic Table Container -->
    <div id="periodic-table-grid" class="periodic-table">
        <!-- Elements will be dynamically generated here -->
    </div>

    <!-- Modal for Detailed Element Information (Hidden by default) -->
    <div id="element-modal" class="fixed inset-0 modal-bg z-50 hidden items-center justify-center transition-opacity duration-300 opacity-0" onclick="closeModal(event)">
        <div id="modal-content" class="modal-content-card rounded-xl shadow-2xl w-full max-w-5xl p-8 transform transition-all duration-500 scale-95" onclick="event.stopPropagation()">
            <button class="absolute top-4 right-4 text-3xl font-light text-gray-400 hover:text-white transition" onclick="closeModal()">
                &times;
            </button>
            
            <div id="modal-body" class="grid md:grid-cols-3 gap-8">
                <!-- Main Element Card (Col 1) -->
                <div id="modal-main-card" class="md:col-span-1 rounded-lg p-6 flex flex-col items-center justify-center text-center border-2" style="background-color: #000000;">
                    <div class="text-8xl font-extrabold" id="modal-symbol">H</div>
                    <div class="text-3xl font-bold mt-2" id="modal-name">Hydrogen</div>
                    <div class="text-lg text-gray-400 mt-1">Atomic Number: <span id="modal-number">1</span></div>
                    <div class="text-xl font-medium mt-4">Mass: <span id="modal-mass">1.008</span></div>
                </div>

                <!-- Details (Col 2 & 3) -->
                <div class="md:col-span-2 text-gray-300">
                    <h2 class="text-2xl font-bold mb-4 text-white">Element Properties</h2>
                    <dl class="space-y-3">
                        <div class="flex justify-between border-b border-gray-700 pb-1">
                            <dt class="font-semibold">Category:</dt>
                            <dd id="modal-category" class="font-light text-right"></dd>
                        </div>
                        <div class="flex justify-between border-b border-gray-700 pb-1">
                            <dt class="font-semibold">Density:</dt>
                            <dd id="modal-density" class="font-light text-right"></dd>
                        </div>
                        <div class="flex justify-between border-b border-gray-700 pb-1">
                            <dt class="font-semibold">Melting Point:</dt>
                            <dd id="modal-melt" class="font-light text-right"></dd>
                        </div>
                        <div class="flex justify-between border-b border-gray-700 pb-1">
                            <dt class="font-semibold">Boiling Point:</dt>
                            <dd id="modal-boil" class="font-light text-right"></dd>
                        </div>

                        <h3 class="text-lg font-semibold pt-4 text-white border-t border-gray-700">Electron Configuration</h3>
                        <div class="flex justify-between pb-1">
                            <dt class="font-medium">Abbreviated:</dt>
                            <dd id="modal-config-abbr" class="font-mono text-sm text-right overflow-x-auto"></dd>
                        </div>
                         <div class="flex justify-between border-b border-gray-700 pb-1">
                            <dt class="font-medium">Full:</dt>
                            <dd id="modal-config-full" class="font-mono text-sm text-right overflow-x-auto"></dd>
                        </div>
                        
                        <div class="flex justify-between border-b border-gray-700 pt-4 pb-1">
                            <dt class="font-semibold">Discovered By:</dt>
                            <dd id="modal-discoverer" class="font-light text-right"></dd>
                        </div>
                        <div class="flex justify-between border-b border-gray-700 pb-1">
                            <dt class="font-semibold">State at STP:</dt>
                            <dd id="modal-state" class="font-light text-right"></dd>
                        </div>
                    </dl>
                    <h3 class="text-xl font-bold mt-6 mb-3 text-white">Summary</h3>
                    <p id="modal-summary" class="text-sm italic text-gray-400"></p>
                </div>
            </div>
        </div>
    </div>


    <script>
        // Data structure includes all 118 elements and separate abbreviated/full configurations
        const elementsData = [
            // Period 1
            { number: 1, symbol: "H", name: "Hydrogen", mass: 1.008, category: "nonmetal", period: 1, group: 1, config: {abbr: "1s¹", full: "1s¹"}, density: "0.08988 g/L", melt: "-259.16 °C", boil: "-252.87 °C", discoverer: "Henry Cavendish", state: "Gas", summary: "The lightest element and the most abundant chemical substance in the universe." },
            { number: 2, symbol: "He", name: "Helium", mass: 4.0026, category: "noble-gas", period: 1, group: 18, config: {abbr: "1s²", full: "1s²"}, density: "0.1786 g/L", melt: "N/A", boil: "-268.93 °C", discoverer: "Pierre Janssen", state: "Gas", summary: "A colorless, odorless, non-toxic, inert, monatomic gas, the second lightest element." },
            // Period 2
            { number: 3, symbol: "Li", name: "Lithium", mass: 6.94, category: "alkali-metal", period: 2, group: 1, config: {abbr: "[He] 2s¹", full: "1s² 2s¹"}, density: "0.534 g/cm³", melt: "180.5 °C", boil: "1342 °C", discoverer: "Johan August Arfwedson", state: "Solid", summary: "A soft, silver-white metal that ignites and burns with a brilliant red color." },
            { number: 4, symbol: "Be", name: "Beryllium", mass: 9.0122, category: "alkaline-earth", period: 2, group: 2, config: {abbr: "[He] 2s²", full: "1s² 2s²"}, density: "1.85 g/cm³", melt: "1287 °C", boil: "2471 °C", discoverer: "Louis Nicolas Vauquelin", state: "Solid", summary: "A strong, lightweight metal, primarily used as a hardening agent in alloys." },
            { number: 5, symbol: "B", name: "Boron", mass: 10.81, category: "metalloid", period: 2, group: 13, config: {abbr: "[He] 2s² 2p¹", full: "1s² 2s² 2p¹"}, density: "2.34 g/cm³", melt: "2076 °C", boil: "3927 °C", discoverer: "Humphry Davy", state: "Solid", summary: "A hard, brittle black element, used in the production of high-strength, lightweight materials." },
            { number: 6, symbol: "C", name: "Carbon", mass: 12.011, category: "nonmetal", period: 2, group: 14, config: {abbr: "[He] 2s² 2p²", full: "1s² 2s² 2p²"}, density: "2.26 g/cm³", melt: "3550 °C", boil: "4027 °C", discoverer: "Ancient", state: "Solid", summary: "The basis of all known life, with allotropes including diamond and graphite." },
            { number: 7, symbol: "N", name: "Nitrogen", mass: 14.007, category: "nonmetal", period: 2, group: 15, config: {abbr: "[He] 2s² 2p³", full: "1s² 2s² 2p³"}, density: "1.251 g/L", melt: "-210.0 °C", boil: "-195.8 °C", discoverer: "Daniel Rutherford", state: "Gas", summary: "A colorless, odorless, unreactive gas that makes up 78% of Earth's atmosphere." },
            { number: 8, symbol: "O", name: "Oxygen", mass: 15.999, category: "nonmetal", period: 2, group: 16, config: {abbr: "[He] 2s² 2p⁴", full: "1s² 2s² 2p⁴"}, density: "1.429 g/L", melt: "-218.4 °C", boil: "-183.0 °C", discoverer: "Joseph Priestley", state: "Gas", summary: "Essential for respiration and combustion, making up about 21% of Earth's atmosphere." },
            { number: 9, symbol: "F", name: "Fluorine", mass: 18.998, category: "halogen", period: 2, group: 17, config: {abbr: "[He] 2s² 2p⁵", full: "1s² 2s² 2p⁵"}, density: "1.696 g/L", melt: "-219.6 °C", boil: "-188.1 °C", discoverer: "Henri Moissan", state: "Gas", summary: "The most chemically reactive and electronegative element, highly toxic." },
            { number: 10, symbol: "Ne", name: "Neon", mass: 20.180, category: "noble-gas", period: 2, group: 18, config: {abbr: "[He] 2s² 2p⁶", full: "1s² 2s² 2p⁶"}, density: "0.9002 g/L", melt: "-248.59 °C", boil: "-246.08 °C", discoverer: "William Ramsay", state: "Gas", summary: "A colorless, odorless, inert gas that glows a bright red-orange when electrified." },
            
            // Period 3
            { number: 11, symbol: "Na", name: "Sodium", mass: 22.990, category: "alkali-metal", period: 3, group: 1, config: {abbr: "[Ne] 3s¹", full: "1s² 2s² 2p⁶ 3s¹"}, density: "0.968 g/cm³", melt: "97.72 °C", boil: "883 °C", discoverer: "Humphry Davy", state: "Solid", summary: "A soft, silver-white, highly reactive metal that floats on water." },
            { number: 12, symbol: "Mg", name: "Magnesium", mass: 24.305, category: "alkaline-earth", period: 3, group: 2, config: {abbr: "[Ne] 3s²", full: "1s² 2s² 2p⁶ 3s²"}, density: "1.738 g/cm³", melt: "650 °C", boil: "1090 °C", discoverer: "Joseph Black", state: "Solid", summary: "A light, structural metal used in alloys and essential for plant and animal life." },
            { number: 13, symbol: "Al", name: "Aluminum", mass: 26.982, category: "post-transition", period: 3, group: 13, config: {abbr: "[Ne] 3s² 3p¹", full: "1s² 2s² 2p⁶ 3s² 3p¹"}, density: "2.70 g/cm³", melt: "660.32 °C", boil: "2519 °C", discoverer: "Hans Christian Ørsted", state: "Solid", summary: "A soft, light, non-magnetic metal with high resistance to corrosion." },
            { number: 14, symbol: "Si", name: "Silicon", mass: 28.085, category: "metalloid", period: 3, group: 14, config: {abbr: "[Ne] 3s² 3p²", full: "1s² 2s² 2p⁶ 3s² 3p²"}, density: "2.3296 g/cm³", melt: "1414 °C", boil: "3265 °C", discoverer: "Jöns Jacob Berzelius", state: "Solid", summary: "The second most abundant element in the Earth's crust, key to the semiconductor industry." },
            { number: 15, symbol: "P", name: "Phosphorus", mass: 30.974, category: "nonmetal", period: 3, group: 15, config: {abbr: "[Ne] 3s² 3p³", full: "1s² 2s² 2p⁶ 3s² 3p³"}, density: "1.82 g/cm³", melt: "44.1 °C", boil: "280.5 °C", discoverer: "Hennig Brand", state: "Solid", summary: "A highly reactive element essential for DNA and bones." },
            { number: 16, symbol: "S", name: "Sulfur", mass: 32.06, category: "nonmetal", period: 3, group: 16, config: {abbr: "[Ne] 3s² 3p⁴", full: "1s² 2s² 2p⁶ 3s² 3p⁴"}, density: "2.07 g/cm³", melt: "115.21 °C", boil: "444.6 °C", discoverer: "Ancient", state: "Solid", summary: "A bright yellow crystalline solid, used primarily in fertilizers and gunpowder." },
            { number: 17, symbol: "Cl", name: "Chlorine", mass: 35.45, category: "halogen", period: 3, group: 17, config: {abbr: "[Ne] 3s² 3p⁵", full: "1s² 2s² 2p⁶ 3s² 3p⁵"}, density: "3.2 g/L", melt: "-101.5 °C", boil: "-34.04 °C", discoverer: "Carl Wilhelm Scheele", state: "Gas", summary: "A highly toxic, greenish-yellow gas, widely used as a disinfectant and bleaching agent." },
            { number: 18, symbol: "Ar", name: "Argon", mass: 39.948, category: "noble-gas", period: 3, group: 18, config: {abbr: "[Ne] 3s² 3p⁶", full: "1s² 2s² 2p⁶ 3s² 3p⁶"}, density: "1.784 g/L", melt: "-189.3 °C", boil: "-185.8 °C", discoverer: "Lord Rayleigh", state: "Gas", summary: "A colorless, odorless, inert gas used in lighting and welding." },
            
            // Period 4 (Including Transition Metals)
            { number: 19, symbol: "K", name: "Potassium", mass: 39.098, category: "alkali-metal", period: 4, group: 1, config: {abbr: "[Ar] 4s¹", full: "[Ne] 3s² 3p⁶ 4s¹"}, density: "0.89 g/cm³", melt: "63.38 °C", boil: "759 °C", discoverer: "Humphry Davy", state: "Solid", summary: "A soft, silver-white metal that reacts violently with water; essential nutrient." },
            { number: 20, symbol: "Ca", name: "Calcium", mass: 40.078, category: "alkaline-earth", period: 4, group: 2, config: {abbr: "[Ar] 4s²", full: "[Ne] 3s² 3p⁶ 4s²"}, density: "1.55 g/cm³", melt: "842 °C", boil: "1484 °C", discoverer: "Humphry Davy", state: "Solid", summary: "A soft, gray alkaline earth metal, essential for bones, teeth, and shells." },
            { number: 21, symbol: "Sc", name: "Scandium", mass: 44.956, category: "transition-metal", period: 4, group: 3, config: {abbr: "[Ar] 3d¹ 4s²", full: "[Ne] 3s² 3p⁶ 3d¹ 4s²"}, density: "2.985 g/cm³", melt: "1541 °C", boil: "2836 °C", discoverer: "Lars Fredrik Nilson", state: "Solid", summary: "A light metal used in minor components for aerospace industry." },
            { number: 22, symbol: "Ti", name: "Titanium", mass: 47.867, category: "transition-metal", period: 4, group: 4, config: {abbr: "[Ar] 3d² 4s²", full: "[Ne] 3s² 3p⁶ 3d² 4s²"}, density: "4.506 g/cm³", melt: "1668 °C", boil: "3287 °C", discoverer: "William Gregor", state: "Solid", summary: "A strong, low-density metal, highly corrosion-resistant, used in aerospace and medical implants." },
            { number: 23, symbol: "V", name: "Vanadium", mass: 50.942, category: "transition-metal", period: 4, group: 5, config: {abbr: "[Ar] 3d³ 4s²", full: "[Ne] 3s² 3p⁶ 3d³ 4s²"}, density: "6.11 g/cm³", melt: "1910 °C", boil: "3407 °C", discoverer: "Andrés Manuel del Río", state: "Solid", summary: "A hard, silvery-gray metal primarily used as an alloying agent for steel." },
            { number: 24, symbol: "Cr", name: "Chromium", mass: 51.996, category: "transition-metal", period: 4, group: 6, config: {abbr: "[Ar] 3d⁵ 4s¹", full: "[Ne] 3s² 3p⁶ 3d⁵ 4s¹"}, density: "7.19 g/cm³", melt: "1857 °C", boil: "2671 °C", discoverer: "Louis Nicolas Vauquelin", state: "Solid", summary: "A steely-grey, lustrous, hard metal known for its bright finish and high melting point." },
            { number: 25, symbol: "Mn", name: "Manganese", mass: 54.938, category: "transition-metal", period: 4, group: 7, config: {abbr: "[Ar] 3d⁵ 4s²", full: "[Ne] 3s² 3p⁶ 3d⁵ 4s²"}, density: "7.21 g/cm³", melt: "1246 °C", boil: "2061 °C", discoverer: "Carl Wilhelm Scheele", state: "Solid", summary: "A brittle, hard metal, often alloyed with steel." },
            { number: 26, symbol: "Fe", name: "Iron", mass: 55.845, category: "transition-metal", period: 4, group: 8, config: {abbr: "[Ar] 3d⁶ 4s²", full: "[Ne] 3s² 3p⁶ 3d⁶ 4s²"}, density: "7.874 g/cm³", melt: "1538 °C", boil: "2862 °C", discoverer: "Ancient", state: "Solid", summary: "The most common element by mass on Earth, essential for steel production." },
            { number: 27, symbol: "Co", name: "Cobalt", mass: 58.933, category: "transition-metal", period: 4, group: 9, config: {abbr: "[Ar] 3d⁷ 4s²", full: "[Ne] 3s² 3p⁶ 3d⁷ 4s²"}, density: "8.90 g/cm³", melt: "1495 °C", boil: "2927 °C", discoverer: "Georg Brandt", state: "Solid", summary: "A hard, ferromagnetic, silver-white metal used in magnetic and high-strength alloys." },
            { number: 28, symbol: "Ni", name: "Nickel", mass: 58.693, category: "transition-metal", period: 4, group: 10, config: {abbr: "[Ar] 3d⁸ 4s²", full: "[Ne] 3s² 3p⁶ 3d⁸ 4s²"}, density: "8.908 g/cm³", melt: "1455 °C", boil: "2913 °C", discoverer: "Axel Fredrik Cronstedt", state: "Solid", summary: "A silvery-white, hard metal, resistant to corrosion and used extensively in alloys
