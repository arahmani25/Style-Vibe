# 🎨 Style Vibe

**AI-Powered Fashion Recommendation System**

A modern fashion discovery platform that uses Large Language Models (LLMs) to understand "vibes" and match users with perfect outfit recommendations from a 10,000+ item dataset.

🔗 **Live Demo:** [https://arahmani25.github.io/Style-Vibe/](https://arahmani25.github.io/Style-Vibe/)

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔍 **Vibe Search** | Natural language fashion search ("casual beach look", "Parisian chic brunch") |
| 📊 **EDA Dashboard** | Interactive analytics with gender, season, and category breakdowns |
| 📸 **Style Scanner** | Upload outfit photos for AI-powered style analysis |
| 🌍 **Trend Radar** | Real-time fashion trend insights powered by Google Search |
| 💼 **My Collection** | Save and organize favorite items |

---

## 🏗️ System Architecture

### Three-Stage Recommendation Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         STYLE VIBE ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   USER QUERY: "casual beach look"                                       │
│                      │                                                  │
│                      ▼                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │  STAGE 1: RETRIEVAL                                              │  │
│   │  • Tokenizes query into keywords                                 │  │
│   │  • Scores 10,000+ items using semantic matching                  │  │
│   │  • Returns top 50 candidates                                     │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                      │                                                  │
│                      ▼                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │  STAGE 2: LLM RE-RANKING                                         │  │
│   │  • Gemini 2.5 Flash analyzes candidates                          │  │
│   │  • Understands aesthetic, occasion, and style                    │  │
│   │  • Re-ranks based on "vibe match"                                │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                      │                                                  │
│                      ▼                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │  STAGE 3: GENERATION                                             │  │
│   │  • Produces Top 3 recommendations                                │  │
│   │  • Generates natural language explanations                       │  │
│   │  • Creates stylist summary                                       │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Frontend** | React 18 + TypeScript | Component-based UI |
| **Styling** | Tailwind CSS | Utility-first styling |
| **Build Tool** | Vite | Fast development and bundling |
| **AI/LLM** | Google Gemini 2.5 Flash | Ranking, analysis, generation |
| **Storage** | IndexedDB (idb) | Client-side caching of 10k items |
| **Hosting** | GitHub Pages | Static site deployment |

---

## 📁 Project Structure

```
stylyst/
├── components/
│   ├── VibeSearch.tsx       # Main search interface
│   ├── Studio.tsx           # Style Scanner + Trends
│   ├── EDA_Dashboard.tsx    # Analytics visualizations
│   ├── InventoryManager.tsx # Dataset management
│   └── ApiKeyModal.tsx      # API configuration
├── services/
│   ├── geminiService.ts     # All LLM integrations
│   ├── storageService.ts    # IndexedDB operations
│   └── githubInventoryService.ts # GitHub data loading
├── App.tsx                  # Main application
├── types.ts                 # TypeScript interfaces
├── constants.ts             # LLM system prompts
├── style.csv                # Fashion dataset metadata
└── images/                  # 10,000+ product images
```

---

## 🧠 Core Components Explained

### 1. Vibe Search (`VibeSearch.tsx`)

The main discovery feature that transforms natural language queries into fashion recommendations.

**Process Flow:**
1. User enters query (e.g., "boho summer festival")
2. Query is tokenized and normalized
3. Each inventory item is scored based on keyword matches
4. Top 50 candidates are sent to Gemini LLM
5. LLM re-ranks and generates explanations
6. Top 3 results displayed with "Why it works" reasoning

**Key Code:**
```typescript
// Local scoring algorithm
tokens.forEach(token => {
    if (item.category.includes(token)) score += 80;
    if (item.name.includes(token)) score += 20;
    if (item.description.includes(token)) score += 5;
});

// LLM re-ranking
const response = await ai.models.generateContent({
    model: 'gemini-2.5-flash',
    systemInstruction: STYLYST_SYSTEM_PROMPT,
    contents: `User wants: "${query}". Rank these candidates...`
});
```

---

### 2. EDA Dashboard (`EDA_Dashboard.tsx`)

Interactive analytics dashboard showing dataset insights.

**Visualizations:**
- 📊 **KPI Cards**: Total items, data quality, categories
- 👤 **Gender Distribution**: Donut chart (Men/Women/Unisex/Kids)
- 🌤️ **Season Breakdown**: Horizontal bar chart
- 📈 **Category Analysis**: Top 8 categories by volume
- 🎨 **Color Palette**: Most common colors extracted
- 🧠 **FAISS Visualization**: t-SNE projection of embeddings

**Data Extraction:**
```typescript
// Gender analysis from descriptions
if (desc.includes('women') || desc.includes('ladies')) {
    genderCounts['Women']++;
}

// Season detection
if (desc.includes('summer') || desc.includes('beach')) {
    seasonCounts['Summer']++;
}
```

---

### 3. Style Scanner (`Studio.tsx`)

AI-powered outfit analysis using Gemini's vision capabilities.

**How it works:**
1. User uploads outfit photo
2. Image converted to base64
3. Sent to Gemini with analysis prompt
4. Returns: Main vibe + individual items detected

**Key Code:**
```typescript
const analyzeImageWithAnnotations = async (imageFile: File) => {
    const base64 = await fileToBase64(imageFile);
    const response = await ai.models.generateContent({
        model: 'gemini-2.5-flash',
        contents: [{
            parts: [
                { inlineData: { data: base64, mimeType: 'image/jpeg' } },
                { text: 'Analyze this outfit...' }
            ]
        }]
    });
    return JSON.parse(response.text);
};
```

---

### 4. Trend Radar (`Studio.tsx`)

Real-time fashion trend discovery using Google Search grounding.

**Key Code:**
```typescript
const getFashionTrends = async (query: string) => {
    const response = await ai.models.generateContent({
        model: 'gemini-2.5-flash',
        contents: `Fashion query: ${query}`,
        config: {
            tools: [{ googleSearch: {} }]  // Real-time web search
        }
    });
    return response.text;
};
```

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn
- Gemini API Key (from [Google AI Studio](https://aistudio.google.com/apikey))

### Installation

```bash
# Clone the repository
git clone https://github.com/arahmani25/Style-Vibe.git
cd Style-Vibe

# Install dependencies
npm install

# Create environment file
echo "VITE_GEMINI_API_KEY=your_api_key_here" > .env.local

# Start development server
npm run dev
```

### Build for Production

```bash
npm run build
npm run preview  # Test production build locally
```

---

## 📊 Dataset

The application uses the **Fashion Product Images (Small)** dataset from Kaggle, containing:

- **10,000+ fashion items**
- Product images (400x500px)
- Metadata: name, category, gender, season, color
- CSV file with product descriptions

**Dataset Structure:**
```
id,description,display name,category
1163.jpg,"Blue round neck team jersey...",India Jersey,Topwear
1541.jpg,"White sports shoes with...",Sports Shoes,Footwear
...
```

---

## 🔐 API Configuration

The app requires a **Google Gemini API key** for AI features.

**Local Development:**
Create `.env.local`:
```
VITE_GEMINI_API_KEY=AIzaSy...
```

**Production:**
The demo uses a hardcoded key for convenience. For production apps, implement a backend proxy.

---

## 📱 Screenshots

### Discover Tab
*Natural language fashion search with AI-powered recommendations*

### EDA Dashboard
*Interactive analytics with charts and visualizations*

### Studio
*Style Scanner and Trend Radar in one place*

---

## 🛠️ Development

### Key Files for Modification

| File | What it controls |
|------|------------------|
| `services/geminiService.ts` | All LLM prompts and API calls |
| `constants.ts` | System prompt for the stylist |
| `components/EDA_Dashboard.tsx` | Analytics and charts |
| `vite.config.ts` | Build configuration |

### Adding New Features

1. **New AI Feature:** Add function in `geminiService.ts`
2. **New Chart:** Add to `EDA_Dashboard.tsx`
3. **New Tab:** Modify `App.tsx` and create component

---

## 📄 License

This project is for educational purposes.

---

## 🙏 Acknowledgments

- **Google Gemini** for LLM capabilities
- **Kaggle** for the fashion dataset
- **React & Vite** communities

---

**Built with ❤️ using React, TypeScript, and Gemini AI**
