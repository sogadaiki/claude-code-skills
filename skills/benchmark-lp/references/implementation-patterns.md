# Implementation Patterns (Phase 4 詳細)

## プロジェクト構成

**Next.js + Tailwind の場合:**
```
src/
├── app/
│   └── (lp)/
│       └── page.tsx          # メインLP
├── components/
│   └── lp/
│       ├── HeroSection.tsx
│       ├── PainPointsSection.tsx
│       ├── SolutionSection.tsx
│       ├── FeaturesSection.tsx
│       ├── TestimonialsSection.tsx
│       ├── PricingSection.tsx
│       ├── FAQSection.tsx
│       ├── CTASection.tsx
│       └── shared/
│           ├── SectionWrapper.tsx
│           └── AnimatedElement.tsx
├── public/
│   ├── images/
│   │   └── lp/              # DALL-E生成画像
│   └── videos/               # 背景動画
└── styles/
    └── lp-animations.css     # カスタムアニメーション
```

**静的HTML の場合:**
```
lp/
├── index.html
├── styles.css
├── images/
└── videos/
```

## 実装順序

1. **グローバルCSS**: アニメーション定義 + カスタムプロパティ
2. **HeroSection**: 最もインパクトが大きいため最初に
3. **各セクション**: 上から順に実装
4. **レスポンシブ調整**: モバイルファースト
5. **画像最適化**: WebP変換、遅延ロード

## 並列実装（サブエージェント活用）

セクション数が多い場合、独立したセクションはサブエージェントで並列実装:

```
Task(subagent_type="general-purpose", name="hero-impl"):
  HeroSection + DALL-E画像生成

Task(subagent_type="general-purpose", name="features-impl"):
  FeaturesSection + アイコン生成

Task(subagent_type="general-purpose", name="testimonials-impl"):
  TestimonialsSection + CTASection
```

各サブエージェントには以下を渡す:
- Phase 2で確定したデザインシステム（配色、フォント、余白）
- 該当セクションのベンチマーク参照コード
- ビジュアルNG集
- アニメーション定義

## コード品質チェック

実装完了後:
```bash
npm run type-check   # TypeScript型チェック
npm run lint         # ESLint
npm run build        # ビルド確認
```
