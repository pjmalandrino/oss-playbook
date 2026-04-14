# Testing Patterns

> Standalone reference checklist. Used during [code review](../processes/code-review.md) and [quality audits](../processes/audit.md).

## Naming conventions

```
# Backend (pytest)
test_<what>_<condition>_<expected>

test_upload_duplicate_file_returns_409
test_analysis_timeout_marks_job_as_failed
test_chunking_empty_document_returns_empty_list

# Frontend (Vitest)
describe('<Component/Function>')
  it('should <expected behavior> when <condition>')

describe('UploadButton')
  it('should show error toast when file exceeds max size')
  it('should disable button during upload')
```

## Test structure (Arrange-Act-Assert)

```python
# Backend
async def test_upload_duplicate_file_returns_409(client, sample_pdf):
    # Arrange
    await client.post("/api/documents", files={"file": sample_pdf})

    # Act
    response = await client.post("/api/documents", files={"file": sample_pdf})

    # Assert
    assert response.status_code == 409
```

```typescript
// Frontend
it('should show error toast when file exceeds max size', async () => {
  // Arrange
  const largeFile = createMockFile({ size: 100 * 1024 * 1024 })

  // Act
  await wrapper.find('[data-e2e="upload-input"]').trigger('change', { files: [largeFile] })

  // Assert
  expect(toast.error).toHaveBeenCalledWith('File too large')
})
```

## What to test by layer

### Domain (unit tests — fast, no I/O)
- Value object creation and validation
- Business rules and invariants
- Edge cases (empty, null, boundary values)

### Services (integration — may use test doubles)
- Use case orchestration
- Error propagation
- Async job lifecycle (pending → running → done/failed)

### API (HTTP tests — full request/response cycle)
- Status codes for success and error paths
- Input validation (missing fields, wrong types, too large)
- Response schema matches Pydantic DTO
- Authentication/authorization if applicable

### Frontend stores (unit — with mocked HTTP)
- State transitions
- Action side effects
- Getter computations

### Frontend components (component tests)
- Renders correctly with props
- User interactions trigger expected behavior
- Loading/error/empty states
- Accessibility (keyboard navigation, ARIA)

## Anti-patterns to avoid

| Anti-pattern | Better approach |
|--------------|----------------|
| Testing implementation details | Test behavior and outcomes |
| `sleep()` or `delay()` in tests | Use `retry()`, `waitFor()`, or async assertions |
| Shared mutable state between tests | Fresh fixtures per test |
| Asserting on CSS classes | Use `data-e2e` attributes |
| Giant test files (500+ lines) | Split by behavior, colocate with source |
| No assertion in test | Every test must assert something |
| Testing framework internals | Test your code, not Vue/FastAPI |

## E2E test tags

| Tag | Scope | Duration | When |
|-----|-------|----------|------|
| `@smoke` | Health checks | ~30s | Every PR |
| `@regression` | Full API coverage | ~2min | PR to release |
| `@e2e` | Cross-domain workflows | ~5min | PR to release |
| `@critical` | 5 core UI journeys | ~1.5min | CI on main |
| `@ui` | All UI features | ~3min | Local dev |

## Coverage expectations

| Layer | Minimum | Target |
|-------|---------|--------|
| Domain | 90% | 95%+ |
| Services | 80% | 90%+ |
| API endpoints | 80% | 90%+ |
| Frontend stores | 70% | 80%+ |
| Frontend components | 60% | 70%+ |
| E2E | Core journeys covered | All user-facing features |
