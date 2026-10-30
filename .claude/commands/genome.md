---
description: Analysis of genetic data for a specific question
argument-hint: <genetics question>
---

# Genome Analysis Request

Analyze the user's genetic data for the question: **$ARGUMENTS**

## VCF file
The VCF file is in the current directory. It can have any name (*.vcf).

## Algorithm

### 0. Find the VCF file
Use Glob to look for the VCF file:
- `pattern`: `*.vcf`

If several are found, ask the user which one to use.
If none is found, report an error.

### 1. Research the topic
Launch an agent to find relevant SNPs:

```
Task tool:
- subagent_type: "general-purpose"
- model: "sonnet"
- prompt: "Find SNPs (rsIDs) related to the topic: $ARGUMENTS.
  Use WebSearch to search GWAS Catalog, SNPedia and scientific papers.
  Return a list of rsIDs with a description of the effect of each allele (which allele is risk/protective)."
```

### 2. Search the genome
Use Grep to look for the rsIDs found in the VCF file (path from Step 0):
- Join all rsIDs with `|` into one pattern
- Path: the VCF file found in Step 0

### 3. Interpretation
Decode the genotypes:
- `0/0` = REF/REF (homozygous for the reference)
- `0/1` = REF/ALT (heterozygous)
- `1/1` = ALT/ALT (homozygous for the alternative)

### 4. Building the report

IMPORTANT: The report must be visually appealing! Use this template:

```markdown
## [Emoji] [Analysis topic]

### Main results

| SNP | Gene | Your genotype | Risk | Interpretation |
|-----|------|---------------|------|----------------|
| rs... | **GENE** | X/X | ✅/🟡/🔴 | Description |

---

### 📊 What this means

**Your profile: [short description]**

**Strengths:**
- Item with an explanation

**Areas of attention:**
- Item: what it means in practice

---

### 🎯 Practical recommendations

1. **Recommendation**: why it matters
2. **Recommendation**: a specific tip

---

### 📈 Overall assessment

| Aspect | Rating |
|--------|--------|
| ... | 🟢/🟡/🔴 |

---

⚠️ **Disclaimer**: Genetics is probabilities, not destiny. This is an informational analysis, not medical advice.

**Sources:** [SNPedia](https://snpedia.com), [GWAS Catalog](https://www.ebi.ac.uk/gwas/)
```

**Emoji for headings:** 🏃 sport, ⚖️ weight, 🧠 mind, ☕ caffeine, 🍷 alcohol, 🥛 nutrition, 💊 medications, ❤️ heart

**Emoji for risks:** ✅ protective, 🟢 neutral, 🟡 moderate, ⚠️ attention, 🔴 high
