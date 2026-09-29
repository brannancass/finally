# Changes Review
Generated: Tue, Sep 29, 2026 3:02:13 PM

8e81569 stuff

## Changes:
diff --git a/.claude/commands/doc-review.md b/.claude/commands/doc-review.md
new file mode 100644
index 0000000..8b860a9
--- /dev/null
+++ b/.claude/commands/doc-review.md
@@ -0,0 +1 @@
+Review the documentation file in the planning file called $ARGUMENTS and add questions, clafications or feedback to a new section at the end along with any opportunities to simplify.
\ No newline at end of file
diff --git a/planning/PLAN.md b/planning/PLAN.md
index bc1811b..974f63e 100644
--- a/planning/PLAN.md
+++ b/planning/PLAN.md
@@ -454,3 +454,98 @@ The container is designed to deploy to AWS App Runner, Render, or any container
 - Portfolio visualization: heatmap renders with correct colors, P&L chart has data points
 - AI chat (mocked): send a message, receive a response, trade execution appears inline
 - SSE resilience: disconnect and verify reconnection
+
+---
+
+## 13. Documentation Review & Feedback
+
+### Questions & Clarifications Needed
+
+**Market Data & Pricing**
+- ✅ **ANSWERED**: The simulator should dynamically support new tickers when users add them to their watchlist.
+- ✅ **ANSWERED**: When Massive API is down or rate-limited, display an error message to the user.
+- ✅ **ANSWERED**: No price validation is necessary at this time.
+
+**Portfolio & Trading Logic**
+- ✅ **ANSWERED**: Fractional shares should be displayed rounded to two decimal places.
+- ✅ **ANSWERED**: When selling shares results in exactly zero quantity, the position should be removed entirely from the positions table.
+- ✅ **ANSWERED**: No trading restrictions are needed - allow unlimited trading frequency.
+
+**Database & Data Persistence**
+- ✅ **ANSWERED**: Delete old portfolio snapshots after 10 days to prevent indefinite growth.
+- ✅ **ANSWERED**: Do not save chat messages at all - every new chat session should start fresh with no conversation history.
+- ✅ **ANSWERED**: Schema changes should retain all existing data when possible. If data loss is unavoidable, alert the user before making any changes.
+
+**LLM Integration**
+- ✅ **ANSWERED**: No token limits needed for this prototype - rely on LLM provider limits.
+- ✅ **ANSWERED**: Rate limiting is not a concern for this prototype.
+- ✅ **ANSWERED**: Display an error message with details of the problem when LLM response parsing fails.
+
+**Frontend Technical Details**
+- ✅ **ANSWERED**: Use Lightweight Charts for financial data visualization (purpose-built for trading applications with better performance for real-time updates).
+- ✅ **ANSWERED**: Use exponential backoff for SSE reconnection (1s, 2s, 4s, 8s) then fail if still unsuccessful.
+- ✅ **ANSWERED**: Yes - show a "Connection Lost" banner or message when SSE connection fails completely after exponential backoff.
+
+### Opportunities to Simplify
+
+**Reduce Scope for MVP**
+1. **Remove fractional shares**: Round all trades to whole shares to simplify math and UI display
+2. **Simplify portfolio snapshots**: Record only after trades, not every 30 seconds - reduces background task complexity
+3. **Single chart type**: Use only line charts initially, skip the treemap heatmap (complex visualization)
+4. **Fixed watchlist**: Start with the 10 default tickers only, remove add/remove functionality for MVP
+5. **Simplified chat**: Remove auto-execution of trades, make it analysis-only initially
+
+**Architecture Simplifications**
+1. **Remove user_id abstraction**: Since it's single-user, remove all `user_id` columns and related logic
+2. **In-memory portfolio tracking**: Skip `portfolio_snapshots` table, calculate P&L on-demand from `trades` table
+3. **Simpler SSE**: Push price updates only when they change, not on a timer
+4. **Static environment**: Remove environment variable switching, use simulator only for MVP
+
+**Database Simplifications**
+1. **Combine tables**: Merge `positions` calculation into real-time computation from `trades` table
+2. **Remove chat persistence**: Keep chat in memory only (lost on restart) to simplify database schema
+3. **Single seed approach**: Remove lazy initialization, use a simple seed SQL file run on container start
+
+### Technical Concerns & Feedback
+
+**Performance & Scalability**
+- SSE to all clients every 500ms could be inefficient. Consider push-only-on-change approach.
+- In-memory price cache is good for single-user but won't scale. Document this limitation.
+- SQLite concurrent write limitations not addressed (though likely fine for single-user).
+
+**Security & Production Readiness**
+- No input validation specified for API endpoints (especially ticker symbols)
+- No rate limiting mentioned for API endpoints
+- OPENROUTER_API_KEY handling in container needs security consideration (not in logs)
+
+**Error Handling Gaps**
+- Network failure scenarios for Massive API not fully specified
+- LLM timeout/failure scenarios need clearer handling
+- Database corruption/recovery strategies missing
+
+**Development Experience**
+- No mention of development mode (hot reload, separate dev containers)
+- Testing strategy lacks integration test layer between unit and E2E
+- No debugging/logging strategy specified
+
+### Recommended Priorities
+
+**Phase 1 (MVP)**:
+1. Basic watchlist with simulator prices (no add/remove)
+2. Simple buy/sell with whole shares only
+3. Basic portfolio table and total value display
+4. SSE price streaming
+5. Simple line chart for selected ticker
+
+**Phase 2 (Enhanced)**:
+1. Add/remove tickers from watchlist
+2. Fractional share support
+3. Portfolio heatmap visualization
+4. P&L over time chart
+
+**Phase 3 (AI Integration)**:
+1. Chat interface with analysis-only responses
+2. Structured LLM output parsing
+3. Auto-execution of trades via chat
+
+This phased approach would allow for incremental development and early validation of core functionality while deferring the more complex AI and visualization features.
