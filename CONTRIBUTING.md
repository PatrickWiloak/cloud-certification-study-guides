# Contributing to Cloud Certification Study Materials

Thank you for your interest in contributing! This repository helps thousands of people prepare for cloud certifications, and your contributions make it even better.

This project is maintained by Patrick Wiloak, an Ex-AWS Solutions Architect with 10 years in tech and 12 cloud certifications. The goal is to create the most comprehensive, free, open-source certification study resource available - and we're studying together!

## 🎯 How You Can Contribute

### 1. **Report Issues**
- Found a typo or error? [Open an issue](../../issues/new)
- Broken link? Let us know!
- Outdated content? Report it

### 2. **Suggest Improvements**
- Have ideas for new content?
- Know a better way to explain something?
- Want to add a new certification?

### 3. **Submit Pull Requests**
- Fix typos and errors
- Update outdated information
- Add missing documentation links
- Create new study materials

## 📝 Contribution Guidelines

### Documentation Standards

**1. Fact Sheet Format**
- Include exam details (duration, questions, cost, passing score)
- Organize by exam domains with official weightings
- Embed documentation links using this format:
  ```markdown
  **[📖 Link Text](https://official-docs-url)** - Brief description
  ```
- Include exam tips, common scenarios, and study strategies
- Add CLI commands and code examples where relevant

**2. Study Notes Format**
- Use clear headings and subheadings
- Include code examples with proper formatting
- Embed official documentation links
- Add real-world scenarios and use cases
- Keep content exam-focused and practical

**3. Link Guidelines**
- Always link to **official documentation** (AWS, Azure, GCP, Oracle, IBM docs)
- Use the 📖 emoji for documentation links
- Include descriptive link text (not "click here")
- Verify all links work before submitting

### Writing Style

- **Clear and concise:** Focus on exam-relevant content
- **Structured:** Use headings, lists, and tables
- **Practical:** Include CLI commands, code examples, and scenarios
- **Accurate:** Verify technical accuracy with official docs
- **Exam-focused:** Highlight exam tips, common scenarios, and gotchas

## 🔄 Pull Request Process

1. **Fork the repository**
   ```bash
   git clone https://github.com/your-username/cloud-certification-notes.git
   cd cloud-certification-notes
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow the documentation standards above
   - Test all links
   - Check for typos and formatting

4. **Commit with descriptive message**
   ```bash
   git add .
   git commit -m "Add: AWS SAA-C03 networking scenarios"
   ```

5. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Open a Pull Request**
   - Provide clear description of changes
   - Reference any related issues
   - Wait for review and feedback

## ✅ Quality Checklist

Before submitting, ensure:

- [ ] All links point to official documentation
- [ ] No broken links (test them!)
- [ ] Proper markdown formatting
- [ ] Consistent style with existing content
- [ ] No typos or grammatical errors
- [ ] Content is exam-focused and accurate
- [ ] Code examples are tested and work
- [ ] CLI commands are correct

## 📚 Content Structure

When adding new certifications, follow this structure:

```
exams/[provider]/[certification-name]/
├── README.md               # Certification overview
├── fact-sheet.md          # Comprehensive exam guide
├── practice-plan.md       # Study schedule
├── strategy.md           # Study strategy (optional)
├── cram-1p.md            # One-page cram sheet (optional)
└── notes/
    ├── topic1.md
    ├── topic2.md
    └── ...
```

## 🎓 Adding a New Certification

1. Create the directory structure (see above)
2. Write comprehensive fact sheet with embedded links
3. Create detailed notes for each exam domain
4. Include practice plan and study strategy
5. Add CLI commands and code examples
6. Update main README.md to include new certification
7. Submit PR with complete certification materials

## 🤝 Code of Conduct

- Be respectful and professional
- Focus on constructive feedback
- Help create a welcoming community
- Remember: we're all here to learn and help others

## 💡 Need Help?

- **Questions?** [Open an issue](../../issues/new)
- **Stuck?** Check existing issues or ask for guidance
- **Want to chat?** Reach out on [LinkedIn](https://linkedin.com/in/patrickwiloak) or [YouTube](https://youtube.com/@patrickwiloak)

## 🌟 Recognition

Contributors will be:
- Listed in the repository's contributors page
- Mentioned in release notes (for significant contributions)
- Appreciated by thousands of certification candidates!

---

## 📋 Quick Contribution Examples

### Example 1: Fix a Typo
```markdown
# In any .md file, fix typos directly
Old: "Amazone S3 is object storage"
New: "Amazon S3 is object storage"
```

### Example 2: Add Documentation Link
```markdown
**[📖 Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)** - Official S3 user guide
```

### Example 3: Add CLI Example
```bash
# Create an S3 bucket
aws s3 mb s3://my-bucket --region us-east-1

# List buckets
aws s3 ls
```

### Example 4: Add Exam Tip
```markdown
**Exam Tip:** S3 is frequently tested. Know the storage classes
(Standard, Intelligent-Tiering, Glacier) and when to use each one.
```

---

## 📫 Contact

**Patrick Wiloak**
- Website: [patrickwiloak.com](https://patrickwiloak.com)
- YouTube: [@patrickwiloak](https://youtube.com/@patrickwiloak)
- GitHub: [@patrickwiloak](https://github.com/patrickwiloak)

---

**Thank you for helping make cloud certification preparation better for everyone!** 🚀

