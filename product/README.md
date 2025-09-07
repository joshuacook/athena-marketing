# Athena Product Vision & Documentation

This directory contains internal product discussions, decisions, and explorations for the Athena platform. This is our space for thinking through product strategy, architecture, and features before they become public documentation.

## Directory Structure

### `/conversations`
Ongoing discussions about product features, architecture, and strategy. Named by date and topic.

### `/decisions`
Architecture Decision Records (ADRs) and product decisions that have been made. These document the "why" behind our choices.

### `/explorations`
Early-stage ideas, experiments, and "what if" scenarios that we're considering but haven't committed to.

## Current Product Understanding

**Athena is "The Complete AI Data Platform" that provides:**

1. **Unified Data Layer** - Single API for all data needs
2. **Three Core Services:**
   - **Context Service** - Manages both structured data AND AI context/embeddings
   - **LLM Service** - Provides access to AI models with context awareness
   - **Storage Service** - Handles file uploads/downloads with AI understanding

## Open Questions

- What exactly is the "Data Service" and how does it differ from what Context Service already provides?
- How do we best explain the interplay between structured data and AI context?
- What's our story for traditional database operations vs. AI-enhanced operations?

## How to Use This Directory

1. Start conversations in `/conversations` with the format: `YYYY-MM-DD-topic-name.md`
2. When decisions are made, document them in `/decisions` as ADRs
3. Use `/explorations` for brainstorming and early ideas
4. Keep the marketing site as our "north star" - what we're building toward
5. Update this README as our understanding evolves

## Key Principle

The marketing site represents our public vision. This directory is our workshop where we figure out how to get there.