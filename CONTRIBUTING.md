# Contributing to Skills GIS

Thank you for your interest in contributing! This guide outlines how to help improve the project.

## How to Contribute

### Report Bugs
- Open an issue with a clear title and description
- Include steps to reproduce and expected behavior
- Attach sample data or error messages if applicable

### Suggest Improvements
- Use GitHub Issues to propose new features or enhancements
- Explain the use case and expected benefits
- Link to relevant scientific publications if available

### Submit New Skills

New skills must follow the template structure:

1. **Create a folder** in `skills/` with the domain name
2. **Document using the template** in `skills.md` format
3. **Include all required sections:**
   - YAML Frontmatter (metadata and keywords)
   - Objective & Domain
   - Input Requirements with validation rules
   - Workflow (QGIS algorithm dictionary)
   - Output Specifications
   - Troubleshooting guide

4. **Test thoroughly** with multiple datasets and edge cases
5. **Submit a Pull Request** with clear description of the skill

### Improve Documentation

- Fix typos or unclear explanations
- Add examples or use cases
- Enhance troubleshooting sections
- Update broken links or references

## Pull Request Process

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/skill-name`
3. Make your changes
4. Commit with clear messages: `git commit -m "Add: [skill name] workflow"`
5. Push to your fork: `git push origin feature/skill-name`
6. Open a Pull Request with description and justification

## Quality Standards

- All skills must have complete, tested workflows
- Documentation must be clear and scientifically accurate
- Code examples must be functional and follow QGIS conventions
- Validation checks must prevent common errors

## Code of Conduct

- Be respectful and constructive
- Focus on technical merit
- Help others learn and improve

## Questions?

Open a GitHub Issue with the label `question` or `discussion`.

---

**Thank you for contributing!**
