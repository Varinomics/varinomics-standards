# Varinomics Coding Style Guideline

This document defines the coding style standard for Varinomics software products.
It is the canonical baseline for new code, code reviews, and product-specific style
profiles.

## 1. Scope

- This guideline applies to production code, tests, tools, build scripts, examples,
  and documentation snippets unless a section says otherwise.
- Product-specific guidance may add stricter rules, but it must not contradict this
  document.
- Language-specific sections refine the general rules. When a general rule and a
  language-specific rule both apply, the more specific rule wins.

## 2. Requirement Language

- `must` means mandatory.
- `should` means the default expectation. Deviations require a concrete reason.
- `may` means allowed but optional.
- When an example and a rule disagree, the rule wins.

## 3. Universal Rules

### 3.1 Code quality goals

- Code must optimize for correctness, leverage, maintainability, and scanability.
- Readability matters, but it is not the only axis. Do not force verbosity or
  repetition solely to make each line locally obvious.
- Prefer concise abstractions, helpers, templates, and reusable mechanisms when
  they remove real duplication or encode an important invariant.
- Dense or advanced code is acceptable when it captures the right abstraction and
  is documented well enough for future maintenance.
- Names must describe intent, not incidental implementation details.
- Abbreviations should be used only when they are well-established in the domain.

### 3.2 Consistency and local coherence

- Match the established conventions of the surrounding module unless this guideline
  explicitly requires a change.
- Preserve intentional alignment and layout when it improves scanability.
- Style-only refactors are allowed when they improve consistency, enforce this
  guideline, or materially improve visual scanability.
- When practical, keep pure beautification passes separate from behavioral changes.

### 3.3 Product evolution and obsolete code

- Do not keep code paths solely to preserve obsolete behavior unless a product
  explicitly requires that compatibility.
- If an old path exists only because of previous behavior and is no longer needed,
  remove it instead of carrying parallel implementations.
- Prefer changing the current code to the desired behavior over layering
  compatibility scaffolding on top of it.
- Contracts and APIs may be changed freely when that is the right design decision,
  unless compatibility requirements are stated explicitly.
- If changing a contract breaks owned consumers, fix those consumers next rather
  than weakening the contract change to avoid breakage.
- Compiler-visible breakage is often useful because it identifies every affected
  consumer immediately.
- Do not add compatibility shims, adapter overloads, legacy parameters, or dual
  code paths merely to avoid changing callers unless compatibility support is an
  explicit requirement.

### 3.4 Structure and control flow

- Prefer explicit control flow over compact but opaque constructs.
- Prefer early exits for invalid or completed cases when they reduce nesting.
- Each block should have one clear responsibility.
- Remove superseded or duplicate code paths instead of keeping parallel ways to do
  the same thing.

### 3.5 Architecture and integration bias

- Prefer fixing defects at the source of ownership when you control that source,
  rather than adding local workarounds downstream.
- Avoid polling as a default design choice. Prefer event-driven or signal-driven
  designs. Use polling only when there is no viable alternative.
- Avoid hysteresis-based behavior unless simpler and more deterministic approaches
  have been exhausted.
- Strongly discourage introducing new environment-variable dependencies for normal
  product behavior, configuration, discovery, or control flow when a clear
  alternative exists.
- Prefer explicit configuration through code, arguments, config files, or other
  local and inspectable mechanisms.
- Use environment variables only when they are genuinely the right integration
  boundary or when no practical alternative exists.

### 3.6 Comments and documentation

- Comments must explain intent, constraints, or non-obvious behavior.
- Comments must not restate code that is already obvious from the implementation.
- Public interfaces and non-trivial internals should have concise documentation.
- Examples in docs and tests should use realistic values and names.

### 3.7 Tooling and enforcement

- Use automated formatting and linting where available, but tool output does not
  override this guideline.
- If a formatter cannot express a rule in this document, follow the document.

## 4. Language-Specific Profiles

- Each language used in a Varinomics product should have a profile section or a
  product-specific addendum.
- A profile may define naming, layout, API, and tooling rules for that language.
- A product may reference this document plus a small local addendum instead of
  maintaining a separate full style guide.

## 5. C++ Profile

This section defines the default C++ style for Varinomics products.

### 5.1 Naming

- Identifiers must use `snake_case` unless a rule below says otherwise.
- Identifiers must not start or end with `_`.
- File names must be lowercase `snake_case`.

Good:

```cpp
int samples_per_pixel;
void set_viewport_size(QSize size);
```

Bad:

```cpp
int SamplesPerPixel;
int _samples;
```

### 5.2 Type names

- Passive data structs must use lowercase names with a `_t` suffix and be declared
  as `struct`.
- A type may use `_t` only when it is a plain data carrier with no custom
  constructor, destructor, inheritance, virtual behavior, or ownership-heavy
  members such as `std::vector`, `QString`, or `std::function`.
- Behavioral or owning types must use `Capitalized_words`.
- Established product or library prefixes keep their documented capitalization.

Good:

```cpp
struct rect_vertex_t {
    glm::vec4 color;
    glm::vec4 rect;
};

class Plot_renderer
{
    // ...
};
```

### 5.3 Members, statics, and constants

- Instance members must use the `m_` prefix.
- Static data members must use the `s_` prefix.
- Namespace-scope compile-time constants must use the `k_` prefix, be
  `constexpr`, and use `snake_case`.

```cpp
class Plot_renderer
{
public:
    void set_viewport_size(const QSize& size) { m_viewport_size = size; }

private:
    static inline int s_msaa_samples = 8;
    QSize m_viewport_size{};
};

constexpr float k_grid_line_half_px = 0.7f;
```

### 5.4 Namespaces

- Do not use `using namespace` at file scope or class scope.
- A `using namespace` directive may be used inside a function body when it removes
  heavy repetition and the scope remains local.
- Namespace aliases and type aliases at file scope, namespace scope, or class
  scope are allowed when they materially reduce repetition and improve
  scanability.
- Prefer a short local alias for verbose namespaces such as `std::filesystem`
  rather than repeating the fully qualified name throughout a file.
- Prefer explicit qualifiers or a short local namespace alias for long names.

```cpp
namespace algo = vnm::plot::algo;
namespace fs = std::filesystem;
```

### 5.5 Files and includes

- In a `.cpp` file, the first include must be the corresponding header.
- Project headers come next.
- Third-party and system headers come after project headers.
- STL headers come last.
- Do not rely on transitive includes.
- Use `#include "..."` for project headers and `#include <...>` for external
  headers.

```cpp
#include "vnm_plot_renderer.h"
#include "vnm_plot.h"
#include "vnm_plot_layout_calculator.h"
#include <glatter/glatter.h>
#include <gtc/matrix_transform.hpp>
#include <QOpenGLShaderProgram>
#include <vector>
```

### 5.6 Braces and layout

- Function, class, and struct opening braces must go on the next line.
- Lambda and control-statement opening braces must go on the same line when the
  condition fits on one line.
- If a control condition wraps across lines, put the opening brace on the next line.
- `else`, `catch`, and `while` for a `do` loop must start on their own line.
- `else if` must be formatted as `else` followed by `if` on the next line.
- Do not write `else if` on one physical line.
- Do not write `} else`, `} catch`, or `} while` on one physical line.
- A short `if` body may stay on one line when it is isolated and clearly easier to
  scan that way.
- Do not compress a repetitive `if`/`else` ladder into one-line bodies when the
  result reads like a run-on chain. In that case, prefer full multi-line blocks
  even if each body contains only one short statement.

Good:

```cpp
void do_render()
{
    if (ready) {
        render();
    }
    else
    if (recover()) {
        render();
    }
}

if (blah1 && blah2 &&
    blah3 && blah4)
{
    render();
}
```

### 5.7 Short one-line forms

- Trivially short functions may stay on one line.
- Intentional no-op functions may use `{}` on one line.
- Consecutive short helpers may be aligned on one line when that improves scanning.
- Single-line control statements may be used only for a tight, consecutive group of
  short statements. Do not use the one-line form for an isolated guard.

```cpp
double Port_assembly::target_vctrl_voltage() { return m_target_vctrl_voltage; }
void install_unhandled_exception_handler() {}

if (value < vmin) { vmin = value; }
if (value > vmax) { vmax = value; }
```

### 5.8 Wrapped parameter lists, template lists, and initializer lists

- When a function declaration, definition, or call wraps, the preferred form is
  to break immediately after `(`.
- Do not use a hanging-indent style where later parameters are lined up under the
  first parameter or under text from the first line.
- Wrapped parameters may stay compact on one continuation line when the types and
  names remain easy to read.
- If the parameter types are visually dense or hard to scan, put one parameter on
  each following line.
- Vertical alignment of types and names across wrapped parameter lines is allowed
  when it materially improves scanability.
- The closing `)` may either align with the start of the function statement or
  stay on its own final parameter line. Use one of those two forms consistently
  within the local block.
- Apply the same rule to wrapped template parameter lists and template argument
  lists.
- If a template list wraps, the preferred form is to break immediately after
  `<`.
- Do not use a hanging-indent style where later template parameters or arguments
  are lined up under the first argument or under text from the first line.
- Wrapped template lists may stay compact on one continuation line when the
  grouped arguments remain easy to scan.
- If the template arguments are visually dense or semantically separate, prefer
  one argument per line or small scannable groups per line.
- The closing `>` may stay on the final argument line or on its own line. Use
  one of those two forms consistently within the local block.
- Constructor initializer lists must start after a blank line.
- Put `:` on its own line, aligned with the function declaration or definition.
- Put each initializer on its own following line, indented one level.
- Use trailing commas at the end of initializer lines. Do not use leading commas.

```cpp
void update_and_write(
    const std::vector<double>& values,
    const std::vector<bool>& filter = {});

static std::filesystem::path resolve_font_path(
    const font_source_t& source, const std::filesystem::path& font_root, Pdf_font font);

static std::filesystem::path resolve_font_path(
    const font_source_t& source,
    const std::filesystem::path& font_root,
    Pdf_font font);

static std::filesystem::path resolve_font_path(
    const font_source_t&         source,
    const std::filesystem::path& font_root,
    Pdf_font                     font);

static std::filesystem::path resolve_font_path(
    const font_source_t& source,
    const std::filesystem::path& font_root,
    Pdf_font font
);

Example(
    QObject* parent,
    Helper helper)
:
    QObject(parent),
    m_helper(std::move(helper))
{}

using block_t = std::variant<
    paragraph_block_t,
    heading_block_t,
    list_block_t,
    code_block_t,
    table_block_t,
    page_break_block_t>;

using some_type = some_template_type<
    group1_param1_t, group1_param2_t, group1_param3_t,
    group2_param1_t, group2_param2_t, group2_param3_t>;

using some_type = some_template_type<
    group1_param1_t, group1_param2_t, group1_param3_t,
    group2_param1_t, group2_param2_t, group2_param3_t
>;
```

Not allowed:

```cpp
static std::filesystem::path resolve_font_path(const font_source_t& source,
                                               const std::filesystem::path& font_root,
                                               Pdf_font font);

using block_t = std::variant<paragraph_block_t,
                             heading_block_t,
                             list_block_t,
                             code_block_t,
                             table_block_t,
                             page_break_block_t>;
```

### 5.9 Control blocks and switches

- Every `if`, `else`, `for`, `while`, `do`, and `catch` block must use braces.
- `case` labels must be indented one level under `switch`.
- This indentation rule also applies to compact one-line `case ...: return ...;`
  and `case ...: break;` forms.
- Do not place `case` labels flush with the `switch` body indentation.
- For short pure dispatch switches, compact one-line `case ...: return ...;`
  forms are preferred when the arms fit cleanly and remain easy to scan.
- Consecutive compact `case` lines may be vertically aligned when that improves
  scanability.
- When consecutive compact `case ...: return ...;` lines are aligned, pad shorter
  labels so the first token after `:` starts in the same column.
- A short fallthrough label may remain on its own line immediately above a compact
  `default: return ...;` when that is the clearest spelling.
- Every `switch` must include a `default`.

```cpp
if (count == 0) {
    return;
}

switch (style) {
    case Display_style::DOTS: return dots;
    case Display_style::AREA: return area;
    case Display_style::LINE: return line;
    default:                  return line;
}

switch (level) {
    case 1:  return body_size * 0.65;
    case 2:  return body_size * 0.55;
    case 3:  return body_size * 0.45;
    case 4:  return body_size * 0.40;
    default: return body_size * 0.35;
}

switch (style) {
    case Inline_style::BOLD:        return Pdf_font::BOLD;
    case Inline_style::ITALIC:      return Pdf_font::ITALIC;
    case Inline_style::BOLD_ITALIC: return Pdf_font::BOLD_ITALIC;
    case Inline_style::CODE:        return Pdf_font::MONO;
    case Inline_style::NORMAL:
    default:                        return Pdf_font::REGULAR;
}
```

Not allowed:

```cpp
switch (style) {
case Display_style::DOTS: return dots;
case Display_style::AREA: return area;
default: return line;
}

if (ready) {
    render();
}
else {
    recover();
}

if (ready) {
    render();
}
else
if (fallback) {
    recover();
}

if ((c & 0x80) == 0x00) {
    len = 1;
}
else
if ((c & 0xE0) == 0xC0) {
    len = 2;
}
else
if ((c & 0xF0) == 0xE0) {
    len = 3;
}
else
if ((c & 0xF8) == 0xF0) {
    len = 4;
}
```

### 5.10 Spacing and formatting

- The default line-length target is 120 columns. Exceed it only when the result is
  clearly easier to read than the wrapped form.
- Use one space after commas and around binary operators.
- Bind `*` and `&` to the type: `Type* name`, `Type& name`.
- Do not leave trailing whitespace.
- Every file must end with a newline.
- Intentional vertical alignment is allowed and encouraged for short groups of
  related lines when it makes comparison faster to read.
- Do not apply alignment mechanically across large, unstable, or weakly related
  blocks.
- For wrapped expression chains such as stream insertion or other repeated
  operators, continue on the next line with a normal continuation indent by
  default.
- The base expression may stand on its own line only when the chain is highly
  repetitive and the operands form a uniform vertical list.
- Do not use deep hanging indentation that lines continuation operators up under
  a token from the previous line.

```cpp
void ensure_tex(GLuint& tex, int& cur_size);
const char* message;

const auto code   = ws->closeCode();
const auto reason = ws->closeReason();

width  = function(something_1, something_else_2);
height = function(something_2, something_else_1);

image_stream
    << "<< /Type /XObject /Subtype /Image /Width " << image.width_px()
    << " /Height " << image.height_px() << " /ColorSpace "
    << (image.color_components() == 1 ? "/DeviceGray" : "/DeviceRGB")
    << " /BitsPerComponent 8";
```

### 5.11 Macros

- Keep multi-line macro continuations visually tidy.
- Either place each trailing `\` one space after the final token or align the `\`
  characters to the same column within that macro.
- Use one style per macro.

```cpp
#define LOG_AND_RETURN(value)  \
    do {                       \
        LOG_INFO(value);       \
        return (value);        \
    }                          \
    while (false)
```

### 5.12 Functions and APIs

- Function names must use `snake_case`.
- Read-only member functions must be marked `const`.
- Variables must be declared in the narrowest practical scope.
- Prefer `auto` when the type is obvious from the initializer and the declaration
  becomes easier to read.
- Prefer free helpers in an anonymous namespace for translation-unit-local symbols.

```cpp
namespace {
constexpr double k_eps = 1e-6;
}
```

### 5.13 Structs, classes, and ownership boundaries

- Use plain data structs for passive aggregates and result bundles.
- Use classes for behavioral or owning types.
- Large classes with heavy private dependencies should use PIMPL.
- Within a class, order sections as `public`, `protected`, then `private`.
- Group members by subsystem or responsibility rather than by declaration kind alone.

```cpp
struct frame_layout_result_t {
    double usable_width;
    double usable_height;
};

class Plot_renderer
{
public:
    explicit Plot_renderer(const Plot* owner);
    ~Plot_renderer();

private:
    struct Impl;
    std::unique_ptr<Impl> d;
};
```

### 5.14 Constants, literals, and enums

- Compile-time constants must use `constexpr` whenever possible.
- Numeric literals should use explicit suffixes where type matters.
- Enum type names follow class-style capitalization.
- Enum values must use `SCREAMING_SNAKE_CASE`.

```cpp
constexpr int k_msaa_samples = 8;
constexpr float k_pixel_snap = 0.5f;

enum class Display_style
{
    DOTS,
    AREA,
    LINE,
};
```

### 5.15 Error handling and logging

- Use assertions for invariants and programmer errors.
- Use structured logging for non-fatal diagnostics.
- Avoid noisy logging in hot paths.
- Exception-based and status-return-based error handling are both acceptable; each
  module should choose deliberately and remain internally consistent.

### 5.16 Qt and OpenGL profile

- Resource creation and cleanup must follow the relevant Qt thread or render-lifecycle
  rules.
- Validate ranges and prerequisites before issuing draw calls.
- Keep OpenGL state changes local and leave the state machine in a known condition.
- Prefer bounded draws and batching when the rendering path allows it.

### 5.17 Comments and API documentation

- Comments should state what matters and why it matters.
- Public APIs and non-trivial internals should use concise Doxygen-style comments
  when that materially improves discoverability.

```cpp
/**
 * @brief Flush batched rectangles to GPU and draw them in one call.
 * @param pmv Combined projection-model-view matrix.
 */
void flush_rects_buffer(const glm::mat4& pmv);
```

### 5.18 Interfacing with C and C-style APIs

- API conventions win when a C or system API has a documented local style.
- C-style casts are acceptable where they are idiomatic, direct, and semantically
  adequate for the target API or conversion.
- Do not mechanically replace C-style casts with C++ casts just to satisfy
  convention.
- Use a C++ cast when the more specific cast category materially improves safety,
  communicates intent better, or avoids ambiguity.
- Do not churn cast style, sentinel values, or null style without a concrete gain in
  correctness, safety, clarity, or consistency within the module.

## 6. Review Checklist

- Names are descriptive and follow the applicable naming rules.
- The change is focused and does not include unrelated formatting churn.
- Control flow is explicit and fully braced.
- Includes are ordered correctly and are not relying on transitive includes.
- Comments explain intent or constraints instead of restating code.
- Constants, types, and ownership boundaries are clear.
- Module-local consistency is preserved.
