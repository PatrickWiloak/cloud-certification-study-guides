# Practice Questions Template

This template defines the format for practice questions in this repository.

---

## Question Format

Each question should follow this structure:

```markdown
### Question X
**Scenario:** [Brief description of the situation - 1-3 sentences]

A. Option A
B. Option B
C. Option C
D. Option D

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** [Explanation of why the correct answer is right and why other options are wrong - 2-5 sentences]

**Key Concept:** [Link to relevant documentation or notes file]
</details>
```

---

## Example Question

### Question 1
**Scenario:** A company needs to store infrequently accessed data that must be retrieved within milliseconds when needed. The data is accessed about once per quarter but retrieval time is critical for business operations.

A. S3 Standard
B. S3 Standard-IA
C. S3 Glacier Instant Retrieval
D. S3 Glacier Deep Archive

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** S3 Glacier Instant Retrieval is designed for data accessed once per quarter with millisecond retrieval times. S3 Standard-IA requires more frequent access (at least once per month) to be cost-effective. S3 Glacier Deep Archive has retrieval times of 12+ hours. S3 Standard is too expensive for infrequently accessed data.

**Key Concept:** [S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
</details>

---

## Guidelines for Writing Questions

### Scenario-Based Questions
- Focus on real-world situations
- Include specific requirements (cost, performance, compliance, etc.)
- Avoid trick questions - test knowledge, not reading comprehension

### Answer Choices
- Make all options plausible
- Avoid "all of the above" or "none of the above"
- Keep options similar in length and detail
- Only one clearly correct answer

### Explanations
- Explain why the correct answer is right
- Briefly explain why wrong answers are wrong
- Link to official documentation or notes

### Domain Coverage
- Distribute questions across all exam domains
- Weight questions according to exam domain percentages
- Cover both common and edge-case scenarios

---

## File Naming Convention

`[provider]-[certification-name].md`

Examples:
- `aws-cloud-practitioner.md`
- `aws-solutions-architect-associate.md`
- `azure-az-104.md`
- `gcp-cloud-engineer.md`
