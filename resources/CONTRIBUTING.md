# Contributing to Cloud & DevOps Certification Resources

Thank you for your interest in contributing to this repository! This guide will help you understand how to add content, maintain documentation standards, and submit your contributions effectively.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Types of Contributions](#types-of-contributions)
3. [Content Guidelines](#content-guidelines)
4. [Documentation Standards](#documentation-standards)
5. [File Organization](#file-organization)
6. [Markdown Formatting](#markdown-formatting)
7. [Link Format and Standards](#link-format-and-standards)
8. [Exam Notes Structure](#exam-notes-structure)
9. [Resource Guidelines](#resource-guidelines)
10. [Pull Request Process](#pull-request-process)
11. [Code of Conduct](#code-of-conduct)
12. [Getting Help](#getting-help)

---

## Getting Started

### Prerequisites

Before contributing, ensure you have:
- A GitHub account
- Git installed on your local machine
- A text editor or IDE (VS Code, Sublime Text, etc.)
- Basic knowledge of Markdown syntax
- Familiarity with Git workflow (clone, branch, commit, push, PR)

### Fork and Clone

1. Fork this repository to your GitHub account
2. Clone your fork locally:
   ```bash
   git clone https://github.com/YOUR-USERNAME/notes-to-eventually-share-beta.git
   cd notes-to-eventually-share-beta
   ```
3. Add the upstream repository:
   ```bash
   git remote add upstream https://github.com/ORIGINAL-OWNER/notes-to-eventually-share-beta.git
   ```
4. Create a new branch for your contribution:
   ```bash
   git checkout -b feature/your-contribution-name
   ```

---

## Types of Contributions

We welcome the following types of contributions:

### 1. Exam Notes
- New certification exam notes
- Updates to existing exam notes
- Additional practice questions
- Study tips and strategies
- Real exam experiences (no NDA violations)

### 2. Service Comparisons
- New service comparison charts
- Updates to existing comparisons
- Additional cloud providers
- Pricing updates
- Feature comparisons

### 3. Resources
- New study resources
- Course recommendations
- Tool suggestions
- Architecture patterns
- Best practices

### 4. Roadmaps
- New certification roadmaps
- Career path guidance
- Learning paths
- Prerequisites documentation

### 5. Documentation Improvements
- Typo fixes
- Clarity improvements
- Formatting corrections
- Dead link fixes
- Additional examples

### 6. Templates and Tools
- Study templates
- Progress trackers
- Cheat sheets
- Quick reference guides

---

## Content Guidelines

### Accuracy and Quality

1. **Verify Information**: Ensure all technical information is accurate and up-to-date
2. **Cite Sources**: Include links to official documentation when referencing services or features
3. **Current Content**: Focus on current exam versions and active certifications
4. **No NDA Violations**: Never share actual exam questions or content under NDA
5. **Original Work**: Only submit content you've created or have permission to share

### Content Standards

1. **Completeness**: Provide comprehensive coverage of topics
2. **Clarity**: Write clearly and concisely; avoid jargon without explanation
3. **Practical Value**: Include real-world examples and use cases
4. **Objectivity**: Present balanced views, especially in comparisons
5. **Accessibility**: Write for various skill levels; explain complex concepts

### What NOT to Include

- Actual exam questions (NDA violation)
- Copyrighted material without permission
- Outdated certification information
- Personal opinions without factual basis
- Marketing or promotional content
- Unverified rumors or speculation
- Offensive or discriminatory content

---

## Documentation Standards

### File Naming Conventions

Use lowercase with hyphens for all files:
- **Good**: `aws-certified-solutions-architect.md`
- **Bad**: `AWS_Certified_Solutions_Architect.md` or `AWSCertifiedSolutionsArchitect.md`

### Directory Structure

```
/
├── exams/
│   ├── aws/
│   ├── azure/
│   └── gcp/
├── resources/
│   ├── service-comparison-*.md
│   ├── certification-roadmap-*.md
│   └── architecture-patterns/
└── README.md
```

### Metadata

Include metadata at the top of exam notes:

```markdown
# AWS Certified Solutions Architect - Associate (SAA-C03)

**Exam Code**: SAA-C03
**Last Updated**: 2024-10-13
**Exam Version**: C03
**Passing Score**: 720/1000
**Duration**: 130 minutes
**Number of Questions**: 65
**Question Types**: Multiple choice, multiple response
**Cost**: $150 USD
```

---

## Markdown Formatting

### Headers

Use proper header hierarchy:

```markdown
# Main Title (H1) - Use once per document
## Major Section (H2)
### Subsection (H3)
#### Detail Level (H4)
```

### Lists

**Unordered Lists**:
```markdown
- Item one
- Item two
  - Nested item
  - Another nested item
- Item three
```

**Ordered Lists**:
```markdown
1. First step
2. Second step
3. Third step
```

### Code Blocks

Use triple backticks with language identifier:

````markdown
```bash
aws s3 ls s3://bucket-name
```

```python
import boto3
s3 = boto3.client('s3')
```

```json
{
  "Version": "2012-10-17",
  "Statement": []
}
```
````

### Tables

Use proper table formatting:

```markdown
| Service | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Compute | EC2 | Virtual Machines | Compute Engine |
| Storage | S3 | Blob Storage | Cloud Storage |
```

### Emphasis

- **Bold**: `**bold text**` for important terms
- *Italic*: `*italic text*` for emphasis
- `Code`: Single backticks for inline code, commands, or service names

### Links

Format links properly:

```markdown
[Link Text](https://example.com)

[AWS Documentation](https://docs.aws.amazon.com/)
```

### Images

If including images (diagrams, screenshots):

```markdown
![Alt Text Description](/path/to/image.png)
```

Store images in an `images/` directory within the relevant section.

---

## Link Format and Standards

### Official Documentation Links

Always link to official documentation for accuracy:

```markdown
**AWS Official Docs**: [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
**Azure Official Docs**: [Virtual Machines Documentation](https://docs.microsoft.com/azure/virtual-machines/)
**GCP Official Docs**: [Compute Engine Documentation](https://cloud.google.com/compute/docs)
```

### Certification Pages

Link to official certification pages:

```markdown
**Certification**: [AWS Certified Solutions Architect - Associate](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
```

### Course and Training Links

Include direct links to courses when recommending:

```markdown
**Recommended Course**: [Stephane Maarek's SAA-C03 Course](https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03/)
```

### Link Validation

Before submitting:
1. Test all links to ensure they're not broken
2. Use HTTPS when available
3. Link to specific sections when possible
4. Avoid linking to paywalled content without noting it
5. Update links if resources move

### Link Freshness

- Check links are current and not outdated
- Remove or update dead links
- Note if a link requires authentication or payment
- Prefer stable, long-term URLs over temporary ones

---

## Exam Notes Structure

When creating new exam notes, follow this structure:

```markdown
# [Certification Name] ([Exam Code])

## Exam Information
- Exam code, duration, cost, passing score
- Prerequisites
- Target audience

## Exam Domains
### Domain 1: [Name] (X% of exam)
#### Topic 1.1
- Key concepts
- Services involved
- Best practices
- Common scenarios

## Key Services
### Service Name
- Overview
- Key features
- Use cases
- Exam tips

## Practice Questions
### Sample Question 1
Question text...

A) Option A
B) Option B
C) Option C
D) Option D

**Answer**: B
**Explanation**: [Detailed explanation]

## Study Resources
- Official documentation
- Recommended courses
- Practice exams
- Additional reading

## Exam Tips
- Time management
- Question-answering strategies
- Common pitfalls
- Last-minute review topics
```

---

## Resource Guidelines

### Service Comparisons

When creating service comparison documents:

1. **Consistent Format**: Use tables for easy comparison
2. **Key Attributes**: Include pricing, features, limits, use cases
3. **Neutral Tone**: Present facts objectively
4. **Current Information**: Verify pricing and features are up-to-date
5. **Citations**: Link to official documentation

Example structure:

```markdown
# Cloud Service Comparison: [Category]

## Overview
Brief introduction to the service category

## Feature Comparison

| Feature | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Service Name | [AWS Service] | [Azure Service] | [GCP Service] |
| Pricing Model | [Details] | [Details] | [Details] |
| Key Features | [List] | [List] | [List] |

## Use Cases
### Use Case 1: [Name]
- AWS: [Approach]
- Azure: [Approach]
- GCP: [Approach]

## Decision Factors
Guidance on choosing between providers

## Links
- [Official AWS Docs]
- [Official Azure Docs]
- [Official GCP Docs]
```

### Certification Roadmaps

When creating roadmap documents:

1. **Clear Progression**: Show logical certification paths
2. **Prerequisites**: Indicate required prior knowledge
3. **Time Estimates**: Provide realistic study time estimates
4. **Resources**: Link to study materials
5. **Career Context**: Explain career benefits

---

## Pull Request Process

### Before Submitting

1. **Review Your Changes**: Read through all modifications
2. **Test Links**: Verify all links work correctly
3. **Check Formatting**: Ensure Markdown renders correctly
4. **Spell Check**: Run a spell checker on your content
5. **Verify Accuracy**: Double-check technical details
6. **Update Metadata**: Set "Last Updated" date if applicable

### Commit Messages

Write clear, descriptive commit messages:

**Good Examples**:
```
Add AWS Solutions Architect Associate exam notes
Update Azure service comparison with new pricing
Fix broken links in GCP certification roadmap
Correct typos in Kubernetes section
```

**Bad Examples**:
```
Update
Fixed stuff
Changes
asdf
```

### Creating a Pull Request

1. Push your branch to your fork:
   ```bash
   git push origin feature/your-contribution-name
   ```

2. Go to the original repository on GitHub

3. Click "New Pull Request"

4. Select your fork and branch

5. Fill out the PR template:

```markdown
## Description
[Describe your changes in detail]

## Type of Change
- [ ] New exam notes
- [ ] Update to existing content
- [ ] New resource/comparison
- [ ] Bug fix (typo, broken link, etc.)
- [ ] Documentation improvement

## Checklist
- [ ] I have read the CONTRIBUTING.md guidelines
- [ ] My content follows the documentation standards
- [ ] I have verified all links work correctly
- [ ] I have tested the Markdown formatting
- [ ] My content is accurate and up-to-date
- [ ] I have not included NDA-protected content
- [ ] I have added appropriate metadata

## Related Issues
[Link any related issues]

## Additional Notes
[Any additional information reviewers should know]
```

### Review Process

1. **Initial Review**: Maintainers will review your PR within 3-5 days
2. **Feedback**: Address any requested changes promptly
3. **Approval**: Once approved, your PR will be merged
4. **Recognition**: Your contribution will be acknowledged

### After Merge

1. Sync your fork with upstream:
   ```bash
   git checkout main
   git pull upstream main
   git push origin main
   ```

2. Delete your feature branch (optional):
   ```bash
   git branch -d feature/your-contribution-name
   ```

---

## Code of Conduct

### Our Standards

- **Be Respectful**: Treat all contributors with respect
- **Be Constructive**: Provide helpful, actionable feedback
- **Be Collaborative**: Work together toward improvements
- **Be Patient**: Remember contributors have varying experience levels
- **Be Inclusive**: Welcome diverse perspectives and backgrounds

### Unacceptable Behavior

- Harassment or discriminatory language
- Personal attacks or insults
- Trolling or inflammatory comments
- Publishing others' private information
- Other conduct deemed unprofessional

### Reporting

Report unacceptable behavior to repository maintainers. All complaints will be reviewed and investigated.

---

## Getting Help

### Questions About Contributing

- Open an issue with the `question` label
- Check existing issues and documentation first
- Be specific about what you need help with

### Technical Issues

If you encounter technical problems:
1. Check GitHub's documentation
2. Search existing issues
3. Open a new issue with detailed information:
   - What you were trying to do
   - What happened instead
   - Steps to reproduce
   - Your environment (OS, Git version, etc.)

### Content Questions

For questions about content:
- Refer to official documentation first
- Ask in an issue if you need clarification
- Maintainers will provide guidance

---

## Style Guide Summary

### Writing Style

1. **Active Voice**: Use active voice when possible
   - Good: "EC2 provides virtual servers"
   - Bad: "Virtual servers are provided by EC2"

2. **Present Tense**: Use present tense
   - Good: "S3 stores objects"
   - Bad: "S3 stored objects"

3. **Second Person**: Address the reader directly
   - Good: "You can use Lambda for serverless compute"
   - Bad: "One can use Lambda for serverless compute"

4. **Concise**: Be clear and concise
   - Good: "Use CloudWatch for monitoring"
   - Bad: "CloudWatch can be utilized for the purpose of monitoring"

### Technical Terms

1. **Consistency**: Use consistent terminology throughout
2. **Capitalization**: Follow official product capitalization
   - AWS Lambda (not AWS lambda or Lambda)
   - Amazon S3 (not Amazon s3 or S3 bucket)
3. **Acronyms**: Define acronyms on first use
   - Good: "Virtual Private Cloud (VPC)"
   - Then use: "VPC"

### Numbers and Units

1. **Numbers**: Spell out one through nine; use numerals for 10+
2. **Units**: Use standard abbreviations (GB, not gigabytes after first use)
3. **Currency**: Use appropriate symbols ($, €, £)

---

## Recognition

Contributors will be recognized in the following ways:

1. **Contributors List**: Added to CONTRIBUTORS.md
2. **Commit History**: Your commits remain in the repository history
3. **Release Notes**: Significant contributions mentioned in release notes
4. **Community Thanks**: Acknowledgment in repository discussions

---

## License

By contributing to this repository, you agree that your contributions will be licensed under the same license as the project (specified in LICENSE file).

---

## Maintenance and Updates

This CONTRIBUTING.md file is maintained by repository owners. If you have suggestions for improving these guidelines:

1. Open an issue describing your proposed changes
2. Discuss with maintainers
3. Submit a PR updating this document once approved

---

## Quick Reference

### New Contributor Checklist

- [ ] Fork and clone repository
- [ ] Create feature branch
- [ ] Make your changes following guidelines
- [ ] Test all links
- [ ] Verify Markdown formatting
- [ ] Write clear commit messages
- [ ] Push to your fork
- [ ] Create pull request with detailed description
- [ ] Address review feedback
- [ ] Celebrate your contribution!

### File Naming Quick Guide

- Lowercase with hyphens: `my-file-name.md`
- Descriptive names: `aws-saa-c03-exam-notes.md`
- Consistent conventions across repository

### Markdown Quick Reference

```markdown
# H1
## H2
### H3

**Bold** *Italic* `Code`

[Link](url)
![Image](path)

- List item
1. Numbered item

| Table | Header |
|-------|--------|
| Cell  | Cell   |

```code block```
```

---

## Thank You!

Thank you for contributing to this project! Your efforts help others succeed in their certification journeys. Every contribution, no matter how small, makes a difference.

**Happy Contributing!**

---

**Contributing Guidelines Version**: 1.0
**Last Updated**: 2024-10-13
**Maintained By**: Repository Owners
