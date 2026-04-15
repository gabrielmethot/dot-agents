## Knowledge Index

Use the `read` tool to load reference files for relevant subjects. If multiple subjects are relevant, load all of them.

Determine subject relevance using these signals (strongest → weakest):

- **Imports (strongest)**: Found in import statements of files in context
- **Terms**: Found in the user’s message or source files
- **Files**: Match currently viewed or edited files

Weak signals should be weighed against context.

### Subjects

**Testing Library**:

- Imports: `@testing-library/*`
- Terms: `testing library`, `test`, `mock`, `render`, `userEvent`, `screen`
- Files: `*.spec.ts`
- References: `references/testing-library.md`

**TypeScript**
- Terms: `typescript`, `typing`
- Files: `tsconfig*.json`, `*.ts`
- References: `references/typescript.md`
