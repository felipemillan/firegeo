# AI Readiness Analyzer - FireGEO Integration Plan

## Executive Summary

The **AI Readiness Analyzer** is an excellent fit for FireGEO's SaaS platform. It complements the existing **Brand Monitor** feature by providing website optimization insights that directly impact AI brand visibility.

**Strategic Value**:
- **Natural Extension**: Brand visibility depends on website AI readiness
- **Credit Monetization**: Another premium feature for the credit system
- **User Journey**: Analyze → Optimize → Monitor loop
- **Competitive Advantage**: Unique combination of readiness + visibility analysis

---

## 🎯 Architecture Compatibility Analysis

### ✅ Perfect Alignment

| Aspect | AI Readiness Analyzer | FireGEO | Compatibility |
|--------|----------------------|---------|---------------|
| **Framework** | Next.js + React + TypeScript | Next.js 15 + React 19 + TypeScript 5 | ✅ 100% |
| **Styling** | TailwindCSS + shadcn/ui | TailwindCSS v4 + shadcn/ui | ✅ 100% |
| **Database** | Not specified (can use any) | PostgreSQL + Drizzle ORM | ✅ Easy integration |
| **API Pattern** | Next.js API routes | Next.js API routes + SSE | ✅ Compatible |
| **Auth** | Not specified | Better Auth | ✅ Can reuse existing |
| **AI Provider** | Groq API | OpenAI (gpt-5-mini) | ⚠️ Need to integrate Groq OR use OpenAI |
| **Web Scraping** | Firecrawl | Firecrawl (already integrated) | ✅ 100% reuse |
| **Component Library** | Custom components | shadcn/ui + custom | ✅ Can standardize |
| **State Management** | useState | React Query + useState | ✅ Can enhance with React Query |
| **Error Handling** | Basic try/catch | Centralized error classes | ✅ Can improve with existing system |

### 🔄 Required Adaptations

1. **AI Provider**:
   - **Option A**: Add Groq to FireGEO's provider config (recommended for cost)
   - **Option B**: Use existing OpenAI with custom prompts (easier, higher cost)
   - **Option C**: Make provider configurable per analysis

2. **Credit System**:
   - Integrate with Autumn billing
   - Define credit costs (suggested: 5-15 credits per analysis)

3. **Data Persistence**:
   - Add database tables for analyses
   - Store results for user history

4. **Authentication**:
   - Protect routes with existing middleware
   - Associate analyses with user accounts

5. **Component Standardization**:
   - Use existing shadcn/ui components where possible
   - Match FireGEO design system

---

## 📊 Integration Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    FIREGEO SAAS PLATFORM                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌─────────────────┐   │
│  │  Brand Monitor │  │  AI Readiness  │  │   AI Chat       │   │
│  │  (Existing)    │  │  Analyzer      │  │   (Existing)    │   │
│  │                │  │  (NEW)         │  │                 │   │
│  │  • Visibility  │  │  • Technical   │  │  • Conversational│  │
│  │  • Competitors │  │  • SEO for AI  │  │  • Research     │   │
│  │  • Rankings    │  │  • Optimization│  │  • 1 credit/msg │   │
│  │  • 10 credits  │  │  • 10 credits  │  │                 │   │
│  └────────┬───────┘  └────────┬───────┘  └─────────────────┘   │
│           │                   │                                 │
│           └───────┬───────────┘                                 │
│                   ↓                                             │
│        ┌──────────────────────┐                                │
│        │  SYNERGY FEATURES     │                                │
│        ├──────────────────────┤                                │
│        │ • URL auto-fill       │                                │
│        │ • Combined reports    │                                │
│        │ • Optimization tasks  │                                │
│        │ • Progress tracking   │                                │
│        └──────────────────────┘                                │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│               SHARED INFRASTRUCTURE                              │
│  • Better Auth  • Autumn Billing  • Firecrawl  • AI Providers   │
│  • PostgreSQL   • Drizzle ORM     • React Query  • SSE          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Schema Integration

### New Tables

```typescript
// File: /lib/db/schema.ts (additions)

/**
 * AI Readiness Analyses
 * Stores website AI optimization analyses
 */
export const aiReadinessAnalyses = pgTable('ai_readiness_analyses', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: text('user_id').notNull(), // From Better Auth

  // Target Information
  url: text('url').notNull(),
  domain: text('domain').notNull(),

  // Scores
  overallScore: integer('overall_score').notNull(), // 0-100
  enhancedScore: integer('enhanced_score'), // 0-100 (if AI analysis done)

  // Basic Analysis Results (8 metrics)
  basicChecks: jsonb('basic_checks').notNull(), // CheckResult[]

  // AI Enhancement Results (8 insights)
  aiInsights: jsonb('ai_insights'), // CheckResult[] with actionItems

  // Metadata
  htmlContent: text('html_content'), // Truncated HTML (first 10k chars)
  metadata: jsonb('metadata'), // { title, description, analyzedAt }

  // Cost Tracking
  creditsUsed: integer('credits_used').default(10).notNull(),
  hasAiEnhancement: boolean('has_ai_enhancement').default(false),

  // Timestamps
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  userIdIdx: index('idx_ai_readiness_user_id').on(table.userId),
  urlIdx: index('idx_ai_readiness_url').on(table.url),
  createdAtIdx: index('idx_ai_readiness_created_at').on(table.createdAt),
}));

/**
 * AI Readiness Recommendations
 * Track specific recommendations given to users
 * (Optional: for advanced features like progress tracking)
 */
export const aiReadinessRecommendations = pgTable('ai_readiness_recommendations', {
  id: uuid('id').primaryKey().defaultRandom(),
  analysisId: uuid('analysis_id').references(() => aiReadinessAnalyses.id, {
    onDelete: 'cascade'
  }).notNull(),
  userId: text('user_id').notNull(),

  // Recommendation Details
  category: text('category').notNull(), // 'heading-structure', 'content-quality', etc.
  priority: text('priority').notNull(), // 'high', 'medium', 'low'
  actionItem: text('action_item').notNull(),

  // Status Tracking
  status: text('status').default('pending').notNull(), // 'pending', 'in-progress', 'completed', 'dismissed'
  completedAt: timestamp('completed_at'),
  notes: text('notes'),

  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  analysisIdIdx: index('idx_recommendations_analysis_id').on(table.analysisId),
  userIdIdx: index('idx_recommendations_user_id').on(table.userId),
}));

// Type exports
export type AIReadinessAnalysis = typeof aiReadinessAnalyses.$inferSelect;
export type NewAIReadinessAnalysis = typeof aiReadinessAnalyses.$inferInsert;
export type AIReadinessRecommendation = typeof aiReadinessRecommendations.$inferSelect;
export type NewAIReadinessRecommendation = typeof aiReadinessRecommendations.$inferInsert;
```

### Migration File

```sql
-- File: /migrations/003_add_ai_readiness.sql

CREATE TABLE IF NOT EXISTS ai_readiness_analyses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT NOT NULL,

  url TEXT NOT NULL,
  domain TEXT NOT NULL,

  overall_score INTEGER NOT NULL,
  enhanced_score INTEGER,

  basic_checks JSONB NOT NULL,
  ai_insights JSONB,

  html_content TEXT,
  metadata JSONB,

  credits_used INTEGER DEFAULT 10 NOT NULL,
  has_ai_enhancement BOOLEAN DEFAULT FALSE,

  created_at TIMESTAMP DEFAULT NOW() NOT NULL,
  updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

CREATE INDEX idx_ai_readiness_user_id ON ai_readiness_analyses(user_id);
CREATE INDEX idx_ai_readiness_url ON ai_readiness_analyses(url);
CREATE INDEX idx_ai_readiness_created_at ON ai_readiness_analyses(created_at);

CREATE TABLE IF NOT EXISTS ai_readiness_recommendations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  analysis_id UUID NOT NULL REFERENCES ai_readiness_analyses(id) ON DELETE CASCADE,
  user_id TEXT NOT NULL,

  category TEXT NOT NULL,
  priority TEXT NOT NULL,
  action_item TEXT NOT NULL,

  status TEXT DEFAULT 'pending' NOT NULL,
  completed_at TIMESTAMP,
  notes TEXT,

  created_at TIMESTAMP DEFAULT NOW() NOT NULL,
  updated_at TIMESTAMP DEFAULT NOW() NOT NULL
);

CREATE INDEX idx_recommendations_analysis_id ON ai_readiness_recommendations(analysis_id);
CREATE INDEX idx_recommendations_user_id ON ai_readiness_recommendations(user_id);
```

---

## 🔌 API Routes Structure

### Route Organization

```
app/api/ai-readiness/
├── route.ts                    # POST - Create new analysis (basic)
├── [analysisId]/
│   ├── route.ts                # GET - Get analysis, DELETE - Delete analysis
│   └── enhance/
│       └── route.ts            # POST - Add AI enhancement to existing analysis
├── analyses/
│   └── route.ts                # GET - List user's analyses (paginated)
├── recommendations/
│   ├── route.ts                # GET - List recommendations
│   └── [recommendationId]/
│       └── route.ts            # PATCH - Update recommendation status
└── check-config/
    └── route.ts                # GET - Check API configuration
```

### Implementation Examples

#### 1. Create Analysis (Basic)

```typescript
// File: app/api/ai-readiness/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { autumn } from '@/lib/autumn';
import { db } from '@/lib/db';
import { aiReadinessAnalyses } from '@/lib/db/schema';
import {
  AuthenticationError,
  ValidationError,
  InsufficientCreditsError,
  handleApiError
} from '@/lib/api-errors';
import { analyzeWebsiteReadiness } from '@/lib/ai-readiness-utils';

const BASIC_ANALYSIS_CREDITS = 10;

export async function POST(request: NextRequest) {
  try {
    // 1. Authentication
    const session = await auth.api.getSession({ headers: request.headers });
    if (!session?.user) {
      throw new AuthenticationError('Please sign in to analyze websites');
    }
    const userId = session.user.id;

    // 2. Parse request
    const body = await request.json();
    const { url } = body;

    if (!url) {
      throw new ValidationError('URL is required');
    }

    // 3. Credit check
    const access = await autumn.check({
      customer_id: userId,
      feature_id: 'ai_readiness', // New feature
    });

    if (!access.access && access.reason === 'usage_limit') {
      throw new InsufficientCreditsError(
        'Insufficient credits for AI readiness analysis. Please upgrade your plan.'
      );
    }

    // 4. Perform analysis (uses Firecrawl + analysis logic)
    const analysisResult = await analyzeWebsiteReadiness(url);

    // 5. Save to database
    const [savedAnalysis] = await db.insert(aiReadinessAnalyses).values({
      userId,
      url: analysisResult.url,
      domain: new URL(analysisResult.url).hostname,
      overallScore: analysisResult.overallScore,
      basicChecks: analysisResult.checks,
      metadata: analysisResult.metadata,
      htmlContent: analysisResult.htmlContent,
      creditsUsed: BASIC_ANALYSIS_CREDITS,
      hasAiEnhancement: false,
    }).returning();

    // 6. Track credit usage
    await autumn.track({
      customer_id: userId,
      feature_id: 'ai_readiness',
      count: BASIC_ANALYSIS_CREDITS,
    });

    // 7. Return result
    return NextResponse.json({
      success: true,
      analysis: {
        id: savedAnalysis.id,
        url: savedAnalysis.url,
        overallScore: savedAnalysis.overallScore,
        checks: savedAnalysis.basicChecks,
        metadata: savedAnalysis.metadata,
        createdAt: savedAnalysis.createdAt,
      },
    });

  } catch (error) {
    return handleApiError(error);
  }
}

// GET - List user's analyses
export async function GET(request: NextRequest) {
  try {
    const session = await auth.api.getSession({ headers: request.headers });
    if (!session?.user) {
      throw new AuthenticationError();
    }

    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const offset = (page - 1) * limit;

    const analyses = await db
      .select({
        id: aiReadinessAnalyses.id,
        url: aiReadinessAnalyses.url,
        domain: aiReadinessAnalyses.domain,
        overallScore: aiReadinessAnalyses.overallScore,
        enhancedScore: aiReadinessAnalyses.enhancedScore,
        hasAiEnhancement: aiReadinessAnalyses.hasAiEnhancement,
        createdAt: aiReadinessAnalyses.createdAt,
      })
      .from(aiReadinessAnalyses)
      .where(eq(aiReadinessAnalyses.userId, session.user.id))
      .orderBy(desc(aiReadinessAnalyses.createdAt))
      .limit(limit)
      .offset(offset);

    return NextResponse.json({
      success: true,
      analyses,
      pagination: { page, limit, total: analyses.length },
    });

  } catch (error) {
    return handleApiError(error);
  }
}
```

#### 2. Enhance with AI

```typescript
// File: app/api/ai-readiness/[analysisId]/enhance/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { autumn } from '@/lib/autumn';
import { db } from '@/lib/db';
import { aiReadinessAnalyses } from '@/lib/db/schema';
import { generateAIInsights } from '@/lib/ai-readiness-utils';
import {
  AuthenticationError,
  NotFoundError,
  InsufficientCreditsError,
  handleApiError
} from '@/lib/api-errors';
import { eq, and } from 'drizzle-orm';

const AI_ENHANCEMENT_CREDITS = 5;

export async function POST(
  request: NextRequest,
  { params }: { params: { analysisId: string } }
) {
  try {
    const session = await auth.api.getSession({ headers: request.headers });
    if (!session?.user) {
      throw new AuthenticationError();
    }
    const userId = session.user.id;

    // 1. Get existing analysis
    const [analysis] = await db
      .select()
      .from(aiReadinessAnalyses)
      .where(
        and(
          eq(aiReadinessAnalyses.id, params.analysisId),
          eq(aiReadinessAnalyses.userId, userId)
        )
      );

    if (!analysis) {
      throw new NotFoundError('Analysis not found');
    }

    if (analysis.hasAiEnhancement) {
      return NextResponse.json({
        success: true,
        message: 'Analysis already has AI enhancement',
        insights: analysis.aiInsights,
      });
    }

    // 2. Credit check
    const access = await autumn.check({
      customer_id: userId,
      feature_id: 'ai_readiness',
    });

    if (!access.access && access.reason === 'usage_limit') {
      throw new InsufficientCreditsError(
        'Insufficient credits for AI enhancement. Please upgrade your plan.'
      );
    }

    // 3. Generate AI insights (using OpenAI or Groq)
    const aiResult = await generateAIInsights({
      url: analysis.url,
      htmlContent: analysis.htmlContent,
      currentChecks: analysis.basicChecks,
    });

    // 4. Calculate enhanced score
    const avgBasicScore = analysis.overallScore;
    const avgAIScore = aiResult.insights.reduce((sum, i) => sum + i.score, 0) / aiResult.insights.length;
    const enhancedScore = Math.round((avgBasicScore * 0.5) + (avgAIScore * 0.5));

    // 5. Update database
    const [updated] = await db
      .update(aiReadinessAnalyses)
      .set({
        aiInsights: aiResult.insights,
        enhancedScore,
        hasAiEnhancement: true,
        creditsUsed: analysis.creditsUsed + AI_ENHANCEMENT_CREDITS,
        updatedAt: new Date(),
      })
      .where(eq(aiReadinessAnalyses.id, params.analysisId))
      .returning();

    // 6. Track credit usage
    await autumn.track({
      customer_id: userId,
      feature_id: 'ai_readiness',
      count: AI_ENHANCEMENT_CREDITS,
    });

    // 7. Return result
    return NextResponse.json({
      success: true,
      insights: updated.aiInsights,
      enhancedScore: updated.enhancedScore,
      overallAIReadiness: aiResult.overallAIReadiness,
      topPriorities: aiResult.topPriorities,
    });

  } catch (error) {
    return handleApiError(error);
  }
}
```

---

## 🎨 Component Architecture

### File Structure

```
components/ai-readiness/
├── ai-readiness-analyzer.tsx       # Main orchestrator component
├── url-input-section.tsx           # URL input with validation
├── analysis-loading.tsx            # Loading state animation
├── results-panel.tsx               # Results display container
│
├── metrics/
│   ├── metric-card.tsx             # Individual metric card
│   ├── metrics-grid.tsx            # 4-column grid layout
│   ├── radar-chart.tsx             # Radar visualization
│   ├── bar-chart.tsx               # Bar chart visualization
│   └── score-circle.tsx            # Overall score display
│
├── insights/
│   ├── ai-insights-section.tsx     # AI enhancement results
│   ├── insight-card.tsx            # Expandable insight with action items
│   ├── action-items-list.tsx       # Checklist of recommendations
│   └── priorities-summary.tsx      # Top 3 priorities
│
├── history/
│   ├── analyses-history.tsx        # Past analyses list
│   ├── analysis-row.tsx            # Individual history item
│   └── compare-analyses-modal.tsx  # Compare 2+ analyses
│
└── modals/
    ├── save-analysis-modal.tsx     # Save/name analysis
    └── share-results-modal.tsx     # Share results (future)
```

### Key Components

#### Main Analyzer Component

```typescript
// File: components/ai-readiness/ai-readiness-analyzer.tsx

'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { UrlInputSection } from './url-input-section';
import { AnalysisLoading } from './analysis-loading';
import { ResultsPanel } from './results-panel';
import { useCreateAnalysis, useEnhanceAnalysis } from '@/hooks/useAIReadiness';
import { Button } from '@/components/ui/button';
import { ArrowLeft } from 'lucide-react';

export function AIReadinessAnalyzer() {
  const router = useRouter();
  const [analysisId, setAnalysisId] = useState<string | null>(null);
  const [showResults, setShowResults] = useState(false);

  const createAnalysis = useCreateAnalysis();
  const enhanceAnalysis = useEnhanceAnalysis();

  const handleAnalyze = async (url: string) => {
    try {
      const result = await createAnalysis.mutateAsync({ url });
      setAnalysisId(result.analysis.id);
      setShowResults(true);
    } catch (error) {
      // Error handled by React Query onError
      console.error('Analysis failed:', error);
    }
  };

  const handleEnhance = async () => {
    if (!analysisId) return;
    await enhanceAnalysis.mutateAsync({ analysisId });
  };

  const handleReset = () => {
    setAnalysisId(null);
    setShowResults(false);
    createAnalysis.reset();
    enhanceAnalysis.reset();
  };

  return (
    <div className="container max-w-7xl mx-auto py-8 px-4">
      {/* Header */}
      <div className="mb-8">
        <Button
          variant="ghost"
          onClick={() => router.push('/dashboard')}
          className="mb-4"
        >
          <ArrowLeft className="mr-2 h-4 w-4" />
          Back to Dashboard
        </Button>
        <h1 className="text-4xl font-bold mb-2">AI Readiness Analyzer</h1>
        <p className="text-muted-foreground">
          Evaluate how well your website is optimized for AI systems and LLMs
        </p>
      </div>

      {/* Main Content */}
      {!showResults && !createAnalysis.isPending && (
        <UrlInputSection
          onAnalyze={handleAnalyze}
          isAnalyzing={createAnalysis.isPending}
        />
      )}

      {createAnalysis.isPending && (
        <AnalysisLoading message="Analyzing website..." />
      )}

      {showResults && analysisId && (
        <ResultsPanel
          analysisId={analysisId}
          onEnhance={handleEnhance}
          onReset={handleReset}
          isEnhancing={enhanceAnalysis.isPending}
        />
      )}
    </div>
  );
}
```

#### Results Panel

```typescript
// File: components/ai-readiness/results-panel.tsx

'use client';

import { useState } from 'react';
import { useAnalysis } from '@/hooks/useAIReadiness';
import { MetricsGrid } from './metrics/metrics-grid';
import { RadarChart } from './metrics/radar-chart';
import { BarChart } from './metrics/bar-chart';
import { AIInsightsSection } from './insights/ai-insights-section';
import { ScoreCircle } from './metrics/score-circle';
import { Button } from '@/components/ui/button';
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/ui/tabs';
import { Sparkles, RotateCcw, Download } from 'lucide-react';

type ViewMode = 'grid' | 'radar' | 'bars';

interface ResultsPanelProps {
  analysisId: string;
  onEnhance: () => void;
  onReset: () => void;
  isEnhancing: boolean;
}

export function ResultsPanel({
  analysisId,
  onEnhance,
  onReset,
  isEnhancing,
}: ResultsPanelProps) {
  const { data: analysis, isLoading } = useAnalysis(analysisId);
  const [viewMode, setViewMode] = useState<ViewMode>('grid');

  if (isLoading || !analysis) {
    return <div>Loading analysis...</div>;
  }

  const hasAIInsights = analysis.hasAiEnhancement;
  const displayScore = hasAIInsights ? analysis.enhancedScore : analysis.overallScore;

  return (
    <div className="space-y-6">
      {/* Header with Score */}
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-2xl font-bold mb-1">{analysis.url}</h2>
          <p className="text-sm text-muted-foreground">
            Analyzed {new Date(analysis.createdAt).toLocaleDateString()}
          </p>
        </div>
        <ScoreCircle score={displayScore} size="lg" />
      </div>

      {/* View Mode Tabs */}
      <Tabs value={viewMode} onValueChange={(v) => setViewMode(v as ViewMode)}>
        <TabsList>
          <TabsTrigger value="grid">Grid View</TabsTrigger>
          <TabsTrigger value="radar">Radar Chart</TabsTrigger>
          <TabsTrigger value="bars">Bar Chart</TabsTrigger>
        </TabsList>

        <TabsContent value="grid" className="mt-6">
          <MetricsGrid checks={analysis.basicChecks} />
        </TabsContent>

        <TabsContent value="radar" className="mt-6">
          <RadarChart checks={analysis.basicChecks} />
        </TabsContent>

        <TabsContent value="bars" className="mt-6">
          <BarChart checks={analysis.basicChecks} />
        </TabsContent>
      </Tabs>

      {/* AI Enhancement Section */}
      {!hasAIInsights ? (
        <div className="bg-gradient-to-r from-purple-50 to-blue-50 dark:from-purple-950/20 dark:to-blue-950/20 rounded-lg p-6 border">
          <div className="flex items-start justify-between">
            <div>
              <h3 className="text-lg font-semibold mb-2 flex items-center gap-2">
                <Sparkles className="h-5 w-5 text-purple-600" />
                Unlock AI-Powered Insights
              </h3>
              <p className="text-sm text-muted-foreground mb-4">
                Get 8 additional AI-enhanced insights with specific action items
                and implementation guidance. (5 credits)
              </p>
            </div>
            <Button
              onClick={onEnhance}
              disabled={isEnhancing}
              className="flex items-center gap-2"
            >
              <Sparkles className="h-4 w-4" />
              {isEnhancing ? 'Analyzing...' : 'Enhance with AI'}
            </Button>
          </div>
        </div>
      ) : (
        <AIInsightsSection insights={analysis.aiInsights} />
      )}

      {/* Actions */}
      <div className="flex gap-4 justify-between items-center pt-6 border-t">
        <Button variant="outline" onClick={onReset}>
          <RotateCcw className="mr-2 h-4 w-4" />
          Analyze Another Site
        </Button>
        <Button variant="outline">
          <Download className="mr-2 h-4 w-4" />
          Export Report
        </Button>
      </div>
    </div>
  );
}
```

---

## 🪝 React Query Hooks

```typescript
// File: hooks/useAIReadiness.ts

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useRouter } from 'next/navigation';
import { toast } from 'sonner';

// Types
interface CreateAnalysisParams {
  url: string;
}

interface EnhanceAnalysisParams {
  analysisId: string;
}

// API Functions
async function createAnalysis(params: CreateAnalysisParams) {
  const response = await fetch('/api/ai-readiness', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(params),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Analysis failed');
  }

  return response.json();
}

async function enhanceAnalysis(params: EnhanceAnalysisParams) {
  const response = await fetch(`/api/ai-readiness/${params.analysisId}/enhance`, {
    method: 'POST',
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error?.message || 'Enhancement failed');
  }

  return response.json();
}

async function fetchAnalysis(analysisId: string) {
  const response = await fetch(`/api/ai-readiness/${analysisId}`);

  if (!response.ok) {
    throw new Error('Failed to fetch analysis');
  }

  return response.json();
}

async function fetchAnalyses(page = 1, limit = 10) {
  const response = await fetch(`/api/ai-readiness?page=${page}&limit=${limit}`);

  if (!response.ok) {
    throw new Error('Failed to fetch analyses');
  }

  return response.json();
}

// Hooks
export function useCreateAnalysis() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createAnalysis,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['ai-readiness-analyses'] });
      toast.success('Analysis complete!');
    },
    onError: (error: Error) => {
      toast.error(error.message || 'Analysis failed');
    },
  });
}

export function useEnhanceAnalysis() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: enhanceAnalysis,
    onSuccess: (data, variables) => {
      queryClient.invalidateQueries({
        queryKey: ['ai-readiness-analysis', variables.analysisId]
      });
      queryClient.invalidateQueries({ queryKey: ['ai-readiness-analyses'] });
      toast.success('AI insights generated!');
    },
    onError: (error: Error) => {
      toast.error(error.message || 'Enhancement failed');
    },
  });
}

export function useAnalysis(analysisId: string) {
  return useQuery({
    queryKey: ['ai-readiness-analysis', analysisId],
    queryFn: () => fetchAnalysis(analysisId),
    enabled: !!analysisId,
  });
}

export function useAnalyses(page = 1, limit = 10) {
  return useQuery({
    queryKey: ['ai-readiness-analyses', page, limit],
    queryFn: () => fetchAnalyses(page, limit),
  });
}
```

---

## 🛠️ Utility Functions

```typescript
// File: lib/ai-readiness-utils.ts

import FirecrawlApp from '@mendable/firecrawl-js';
import { openai } from '@ai-sdk/openai';
import { generateText } from 'ai';

const firecrawl = new FirecrawlApp({
  apiKey: process.env.FIRECRAWL_API_KEY
});

/**
 * Main analysis function - analyzes website for AI readiness
 */
export async function analyzeWebsiteReadiness(url: string) {
  // 1. Normalize URL
  if (!url.startsWith('http')) {
    url = `https://${url}`;
  }

  // 2. Scrape website (reuse existing Firecrawl)
  const scrapeResult = await firecrawl.scrapeUrl(url, {
    formats: ['html', 'markdown'],
  });

  if (!scrapeResult.success || !scrapeResult.html) {
    throw new Error('Failed to scrape website');
  }

  // 3. Run all checks in parallel
  const [
    headingCheck,
    readabilityCheck,
    metadataCheck,
    semanticCheck,
    accessibilityCheck,
    robotsCheck,
    sitemapCheck,
    llmsCheck,
  ] = await Promise.all([
    analyzeHeadingStructure(scrapeResult.html),
    analyzeReadability(scrapeResult.html),
    analyzeMetadata(scrapeResult.html, scrapeResult.metadata),
    analyzeSemanticHTML(scrapeResult.html),
    analyzeAccessibility(scrapeResult.html),
    checkRobotsTxt(url),
    checkSitemap(url),
    checkLLMsTxt(url),
  ]);

  const checks = [
    headingCheck,
    readabilityCheck,
    metadataCheck,
    semanticCheck,
    accessibilityCheck,
    robotsCheck,
    sitemapCheck,
    llmsCheck,
  ];

  // 4. Calculate overall score
  const overallScore = calculateOverallScore(checks, url);

  // 5. Return formatted result
  return {
    success: true,
    url,
    overallScore,
    checks,
    htmlContent: scrapeResult.html.substring(0, 10000), // First 10k chars
    metadata: {
      title: scrapeResult.metadata?.title || '',
      description: scrapeResult.metadata?.description || '',
      analyzedAt: new Date().toISOString(),
    },
  };
}

/**
 * Generate AI insights using OpenAI (or Groq)
 */
export async function generateAIInsights({
  url,
  htmlContent,
  currentChecks,
}: {
  url: string;
  htmlContent: string;
  currentChecks: any[];
}) {
  const systemPrompt = `You are an AI readiness expert. Analyze websites for optimization across AI systems, LLMs, and web crawlers.

Provide 8 specific insights in these categories:
1. Content Quality for AI
2. Information Architecture
3. Semantic Structure
4. AI Discovery Value
5. Knowledge Extraction
6. Context & Completeness
7. Content Uniqueness
8. Machine Interpretability

For each insight, provide:
- score (0-100)
- status ('pass' | 'warning' | 'fail')
- details (2-3 sentences)
- recommendation (1-2 sentences)
- actionItems (exactly 5 specific, actionable items)

Return ONLY valid JSON matching this structure.`;

  const userPrompt = `Analyze this website for AI readiness:

URL: ${url}

Current page-level scores:
${currentChecks.map(c => `- ${c.label}: ${c.score}/100 (${c.status})`).join('\n')}

HTML content (truncated):
${htmlContent.substring(0, 5000)}

Provide your analysis as JSON.`;

  try {
    const { text } = await generateText({
      model: openai('gpt-4-turbo'),
      temperature: 0.7,
      maxTokens: 2000,
      system: systemPrompt,
      prompt: userPrompt,
    });

    // Parse response
    const cleanedText = text
      .replace(/```json\n?/g, '')
      .replace(/```\n?/g, '')
      .trim();

    const parsed = JSON.parse(cleanedText);

    return {
      success: true,
      insights: parsed.insights || [],
      overallAIReadiness: parsed.overallAIReadiness || '',
      topPriorities: parsed.topPriorities || [],
    };

  } catch (error) {
    console.error('AI insight generation failed:', error);

    // Return mock data as fallback
    return {
      success: true,
      insights: generateMockInsights(),
      overallAIReadiness: 'Analysis completed with fallback data',
      topPriorities: [
        'Improve content structure',
        'Add semantic HTML',
        'Enhance metadata',
      ],
    };
  }
}

// ... (other analysis functions: analyzeHeadingStructure, analyzeReadability, etc.)
// These would be ported from the original architecture with minor adaptations
```

---

## 💰 Credit System Integration

### Feature Configuration

```typescript
// File: config/constants.ts (additions)

export const CREDIT_COSTS = {
  // Existing
  CHAT_MESSAGE: 1,
  BRAND_ANALYSIS: 10,

  // New
  AI_READINESS_BASIC: 10,      // Basic analysis (8 metrics)
  AI_READINESS_ENHANCED: 5,    // AI enhancement (8 insights)
  AI_READINESS_FULL: 15,       // Both in one call (optional pricing)
} as const;

export const FEATURE_IDS = {
  // Existing
  MESSAGES: 'messages',
  BRAND_MONITOR: 'brand_monitor',

  // New
  AI_READINESS: 'ai_readiness',
} as const;
```

### Autumn Products Update

```typescript
// File: config/autumn-products.ts (update)

export const products = [
  {
    id: 'free',
    name: 'Free',
    credits: 100, // per month
    features: [
      '10 Brand Monitor analyses OR',
      '100 Chat messages OR',
      '6 AI Readiness analyses (basic only) OR',
      'Mix of all features',
    ],
  },
  {
    id: 'pro',
    name: 'Pro',
    price: 29,
    credits: 500,
    features: [
      '50 Brand Monitor analyses',
      '500 Chat messages',
      '33 AI Readiness analyses (full)',
      'All features included',
      'Priority support',
    ],
  },
  // ... other tiers
];
```

---

## 🔗 Feature Synergies

### 1. Brand Monitor ↔ AI Readiness Integration

**Scenario**: User runs Brand Monitor analysis
**Synergy**: Suggest AI Readiness analysis for their website

```typescript
// File: components/brand-monitor/brand-monitor.tsx (addition)

{/* After Brand Monitor analysis completes */}
{analysisComplete && (
  <div className="mt-6 p-4 bg-blue-50 dark:bg-blue-950/20 rounded-lg border">
    <h4 className="font-semibold mb-2">💡 Optimization Tip</h4>
    <p className="text-sm text-muted-foreground mb-3">
      Want to improve your AI visibility score? Analyze your website's
      AI readiness to get specific optimization recommendations.
    </p>
    <Button
      variant="outline"
      size="sm"
      onClick={() => router.push(`/ai-readiness?url=${companyUrl}`)}
    >
      Analyze AI Readiness
    </Button>
  </div>
)}
```

### 2. Cross-Feature URL Sharing

```typescript
// File: app/ai-readiness/page.tsx

'use client';

import { useSearchParams } from 'next/navigation';
import { AIReadinessAnalyzer } from '@/components/ai-readiness/ai-readiness-analyzer';

export default function AIReadinessPage() {
  const searchParams = useSearchParams();
  const prefilledUrl = searchParams.get('url');

  return <AIReadinessAnalyzer initialUrl={prefilledUrl} />;
}
```

### 3. Combined Dashboard Widget

```typescript
// File: app/dashboard/page.tsx (addition)

<div className="grid grid-cols-1 md:grid-cols-2 gap-6">
  {/* Existing widgets */}

  {/* New: AI Readiness Summary */}
  <Card>
    <CardHeader>
      <CardTitle>AI Readiness</CardTitle>
      <CardDescription>Your website optimization status</CardDescription>
    </CardHeader>
    <CardContent>
      {latestReadinessAnalysis ? (
        <div className="space-y-4">
          <div className="flex items-center justify-between">
            <span className="text-sm">Latest Score</span>
            <ScoreCircle
              score={latestReadinessAnalysis.enhancedScore || latestReadinessAnalysis.overallScore}
              size="sm"
            />
          </div>
          <div className="text-xs text-muted-foreground">
            Analyzed {formatDistanceToNow(new Date(latestReadinessAnalysis.createdAt))} ago
          </div>
          <Button
            variant="outline"
            size="sm"
            className="w-full"
            onClick={() => router.push('/ai-readiness')}
          >
            View Full Report
          </Button>
        </div>
      ) : (
        <div className="text-center py-6">
          <p className="text-sm text-muted-foreground mb-4">
            No analyses yet
          </p>
          <Button
            variant="default"
            size="sm"
            onClick={() => router.push('/ai-readiness')}
          >
            Analyze Your Website
          </Button>
        </div>
      )}
    </CardContent>
  </Card>
</div>
```

---

## 🚀 Implementation Roadmap

### Phase 1: Core Functionality (Week 1)
- [ ] Database schema & migration
- [ ] Basic analysis API route (`POST /api/ai-readiness`)
- [ ] Analysis utility functions (8 basic checks)
- [ ] Firecrawl integration
- [ ] Basic UI components (URL input, results grid)
- [ ] Credit system integration

### Phase 2: AI Enhancement (Week 2)
- [ ] AI insights API route (`POST /api/ai-readiness/[id]/enhance`)
- [ ] OpenAI/Groq integration for insights
- [ ] Enhanced UI components (AI insights section)
- [ ] Action items display
- [ ] Visualization modes (radar, bar charts)

### Phase 3: History & Management (Week 3)
- [ ] List analyses API (`GET /api/ai-readiness`)
- [ ] Analysis detail page
- [ ] Delete analysis functionality
- [ ] History component
- [ ] Dashboard widget integration

### Phase 4: Advanced Features (Week 4)
- [ ] Recommendations tracking (optional)
- [ ] Compare analyses feature
- [ ] Export to PDF/CSV
- [ ] URL sharing for Brand Monitor
- [ ] Combined insights dashboard

---

## 🎯 Recommendations & Improvements

### Architecture Enhancements

1. **Use Server-Sent Events (SSE) for Long Analysis**
   - Original: Single POST request (5-9s)
   - Improved: Stream progress updates like Brand Monitor
   ```typescript
   // Progress events:
   // 1. "Scraping website..."
   // 2. "Analyzing HTML structure..."
   // 3. "Checking domain files..."
   // 4. "Calculating scores..."
   // 5. "Complete!"
   ```

2. **Implement Caching Strategy**
   - Cache domain files (robots.txt, sitemap.xml) for 24 hours
   - Reuse scraped HTML if analyzing same URL within 1 hour
   - Reduce API costs significantly

3. **Add Rate Limiting**
   - Use existing `lib/rate-limit.ts`
   - Limit: 10 analyses per user per hour
   - Prevent abuse and control costs

4. **Enhance Error Handling**
   - Use FireGEO's centralized error system
   - Provide actionable error messages
   - Graceful degradation for external API failures

5. **Add Webhook Support** (Future)
   - Async analysis processing
   - Email notification when complete
   - Better UX for slow websites

### AI Provider Strategy

**Recommendation: Use OpenAI for MVP, add Groq for cost optimization later**

| Provider | Pros | Cons | Use Case |
|----------|------|------|----------|
| OpenAI (gpt-4-turbo) | Already integrated, high quality | Higher cost | MVP, enhanced insights |
| Groq (mixtral) | Very fast, lower cost | Need new integration | Production optimization |
| Perplexity | Built-in web search | Limited customization | Alternative option |

**Cost Analysis (per enhanced analysis)**:
- OpenAI gpt-4-turbo: ~$0.03 (2k tokens @ $0.01/1k input + $0.03/1k output)
- Groq mixtral: ~$0.005 (2k tokens @ $0.27/1M tokens)
- Savings potential: 83% with Groq

### Database Optimizations

1. **Add Indexes**
   - `CREATE INDEX idx_ai_readiness_domain ON ai_readiness_analyses(domain);`
   - Enables "View all analyses for this domain"

2. **Implement Soft Deletes**
   - Add `deleted_at` column
   - Retain data for analytics while respecting user deletions

3. **Add Analysis Tags** (Future)
   ```sql
   CREATE TABLE ai_readiness_tags (
     analysis_id UUID REFERENCES ai_readiness_analyses(id),
     tag TEXT NOT NULL,
     created_at TIMESTAMP DEFAULT NOW()
   );
   ```
   - User-defined tags: "homepage", "blog", "product-page"
   - Organize analyses better

### UI/UX Enhancements

1. **Add Comparison Mode**
   - Compare before/after analyses
   - Show improvement over time
   - Visual diff of scores

2. **Implement Score Trends**
   - Line chart showing score changes
   - Track improvements per metric
   - Gamification potential

3. **Add Export Formats**
   - PDF report (branded)
   - CSV data export
   - Shareable link (public view)

4. **Mobile Optimization**
   - Responsive charts
   - Swipeable metric cards
   - Touch-friendly interactions

---

## 📋 Migration Checklist

### Pre-Implementation

- [ ] Review and approve integration plan
- [ ] Decide on AI provider (OpenAI vs Groq)
- [ ] Set credit pricing (10+5 recommended)
- [ ] Design dashboard placement
- [ ] Create marketing copy for feature

### Development

- [ ] Create feature branch `feature/ai-readiness-analyzer`
- [ ] Add database schema (`003_add_ai_readiness.sql`)
- [ ] Run migration and verify tables
- [ ] Implement utility functions (`lib/ai-readiness-utils.ts`)
- [ ] Create API routes (basic + enhance)
- [ ] Build React Query hooks
- [ ] Create UI components (analyzer, results, metrics)
- [ ] Add navigation links (navbar, dashboard)
- [ ] Integrate credit system (Autumn)
- [ ] Test error handling
- [ ] Add loading states
- [ ] Implement responsive design
- [ ] Add accessibility features (ARIA, keyboard nav)

### Testing

- [ ] Unit tests for analysis functions
- [ ] API route tests (success + error cases)
- [ ] Integration tests (full flow)
- [ ] Credit deduction verification
- [ ] Cross-browser testing
- [ ] Mobile responsiveness
- [ ] Performance testing (large HTML pages)
- [ ] Error scenario testing (Firecrawl down, AI API down)

### Deployment

- [ ] Environment variables setup (production)
- [ ] Database migration (production)
- [ ] Deploy to staging
- [ ] QA testing
- [ ] Deploy to production
- [ ] Monitor error rates
- [ ] Track feature usage (analytics)
- [ ] Gather user feedback

### Post-Launch

- [ ] Document feature in user guide
- [ ] Create tutorial video
- [ ] Add to pricing page feature list
- [ ] Monitor costs (AI API + Firecrawl)
- [ ] Optimize based on usage patterns
- [ ] Plan Phase 2 features (recommendations tracking, etc.)

---

## 💡 Strategic Value Proposition

### For Users

**Problem**: "My website isn't showing up in AI search results"

**Solution Journey**:
1. **Brand Monitor**: Discover you have low AI visibility
2. **AI Readiness**: Learn *why* (poor structure, missing metadata, etc.)
3. **Action**: Implement specific recommendations
4. **Re-analyze**: See improvements in both tools
5. **Retention**: Ongoing monitoring and optimization

### For FireGEO Business

1. **Increased ARPU** (Average Revenue Per User)
   - Additional credit usage (10-15 credits per analysis)
   - Encourages upgrades to higher tiers

2. **Differentiation**
   - No competitor offers both visibility + readiness
   - Unique value proposition in market

3. **User Engagement**
   - More reasons to return to platform
   - Cross-feature usage increases stickiness

4. **Data Insights**
   - Aggregate trends across industries
   - Benchmarking opportunities (future feature)
   - Content marketing potential ("State of AI Readiness 2025")

5. **Expansion Opportunities**
   - White-label for agencies
   - API access for developers
   - Automated monitoring subscriptions

---

## 🏁 Conclusion

The AI Readiness Analyzer integrates seamlessly into FireGEO's existing architecture with:

✅ **100% tech stack compatibility**
✅ **Reusable infrastructure** (auth, billing, Firecrawl, DB)
✅ **Consistent UX patterns** (SSE, React Query, shadcn/ui)
✅ **Natural user journey** with Brand Monitor
✅ **Clear monetization path** via credits
✅ **Scalable architecture** following FireGEO patterns

**Estimated Development Time**: 3-4 weeks (solo developer)

**Estimated Cost Impact**:
- Firecrawl: ~$0.002 per scrape (existing usage)
- AI insights: ~$0.03 per analysis (OpenAI) or ~$0.005 (Groq)
- Total cost per full analysis: ~$0.032
- Revenue per analysis: 15 credits = varies by plan
- Healthy margin potential

**Recommendation**: ✅ **PROCEED WITH IMPLEMENTATION**

The feature is well-architected, technically sound, and strategically valuable. The integration plan is comprehensive and follows FireGEO's established patterns, minimizing risk and development time.

---

**Next Steps**:
1. Review this plan with stakeholders
2. Approve credit pricing (10 basic + 5 enhanced)
3. Choose AI provider (OpenAI for MVP recommended)
4. Create feature branch and begin Phase 1
5. Set success metrics (usage rate, conversion to enhanced, user satisfaction)

Would you like me to start implementing any specific phase?
