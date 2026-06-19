---
name: rhino-eto-ui
description: |
  Complete guide for building cross-platform Rhino UI with Eto.Forms — dialogs, panels, layouts, controls, and dockable side panels. Use this skill for any Rhino UI development: "eto forms rhino", "rhino dialog c#", "eto dialog", "rhinocommon ui", "tabbed panel rhino", "eto layout", "rhino side panel", "IPanel rhino", "Panels.RegisterPanel", "eto controls", "rhino dark mode ui", "UseRhinoStyle", "DynamicLayout eto", "rhino dockable panel".
---

# Eto.Forms — Rhino Cross-Platform UI

Eto.Forms renders to WPF/WinForms on Windows, Cocoa on macOS — one codebase for both.

## Namespaces

```csharp
using Eto.Forms;
using Eto.Drawing;
using Rhino.UI; // for UseRhinoStyle and Rhino-specific extensions
```

## Modal Dialog (Blocking)

```csharp
public class MyDialog : Dialog<DialogResult>
{
    private TextBox _nameBox;
    private NumericStepper _valueStepper;

    public string InputName => _nameBox.Text;
    public double InputValue => _valueStepper.Value;

    public MyDialog()
    {
        Title = "My Dialog";
        Padding = new Padding(10);
        Resizable = true;
        MinimumSize = new Size(300, 200);

        _nameBox = new TextBox { PlaceholderText = "Enter name..." };
        _valueStepper = new NumericStepper { Value = 1.0, MinValue = 0, MaxValue = 100, Increment = 0.5 };

        var okButton = new Button { Text = "OK" };
        okButton.Click += (s, e) => Close(DialogResult.Ok);

        var cancelButton = new Button { Text = "Cancel" };
        cancelButton.Click += (s, e) => Close(DialogResult.Cancel);

        var layout = new DynamicLayout { Padding = new Padding(5), Spacing = new Size(5, 5) };
        layout.AddRow(new Label { Text = "Name:" }, _nameBox);
        layout.AddRow(new Label { Text = "Value:" }, _valueStepper);
        layout.AddSeparateRow(null, okButton, cancelButton); // null = expander (pushes right)

        Content = layout;
        this.UseRhinoStyle();
        DefaultButton = okButton;
        AbortButton = cancelButton;
    }
}

// Usage in a Command:
var dlg = new MyDialog();
if (dlg.ShowModal(RhinoEtoApp.MainWindow) == DialogResult.Ok)
{
    string name = dlg.InputName;
    double value = dlg.InputValue;
}
```

## Non-Modal Form (Non-Blocking)

```csharp
public class MyToolWindow : Form
{
    public MyToolWindow()
    {
        Title = "My Tool";
        Size = new Size(250, 400);
        Location = new Point(100, 100);

        // ... build content ...

        this.UseRhinoStyle();
        
        // Prevent close from destroying — hide instead
        Closing += (s, e) =>
        {
            e.Cancel = true;
            Visible = false;
        };
    }
}

// Show/hide (keep alive)
private static MyToolWindow _window;
public static void ShowWindow()
{
    if (_window == null) _window = new MyToolWindow();
    _window.Show();
    _window.BringToFront();
}
```

## Common Controls

```csharp
// Text
var label = new Label { Text = "Label:", TextAlignment = TextAlignment.Right };
var textBox = new TextBox { PlaceholderText = "Enter value" }; // TextBox has PlaceholderText
var textArea = new TextArea { Height = 80, Wrap = true };      // TextArea does NOT have PlaceholderText
var passwordBox = new PasswordBox();

// Numbers
var numericStepper = new NumericStepper { Value = 0, MinValue = -100, MaxValue = 100, Increment = 1, DecimalPlaces = 2 };
var slider = new Slider { MinValue = 0, MaxValue = 100, Value = 50, Orientation = Orientation.Horizontal };

// Boolean
var checkBox = new CheckBox { Text = "Enable feature", Checked = false };
var radioA = new RadioButton { Text = "Option A", Checked = true };
var radioB = new RadioButton(radioA) { Text = "Option B" }; // same group

// Selection
var dropDown = new DropDown();
dropDown.Items.AddRange(new[] { "Item 1", "Item 2", "Item 3" }.Select(s => new ListItem { Text = s }));
dropDown.SelectedIndex = 0;

var listBox = new ListBox { Height = 100 };
listBox.Items.Add("Row 1");

// Buttons
var button = new Button { Text = "Click Me", Size = new Size(80, 25) };
var imageButton = new ImageButton { Image = /* Bitmap */ null };
var toggleButton = new ToggleButton { Text = "Toggle" }; // Checked property for state

// Containers
var groupBox = new GroupBox { Text = "Group Title", Content = innerControl };
var scrollable = new Scrollable { Content = bigControl, ExpandContentWidth = true };
var expander = new Expander { Header = "Advanced", Content = advancedContent };
var tabControl = new TabControl();
tabControl.Pages.Add(new TabPage { Text = "Tab 1", Content = tab1Content });

// File/Color/Folder pickers
var colorPicker = new ColorPicker { Value = Colors.Red };
var filePicker = new FilePicker { FileAction = FileAction.OpenFile };
// Folder picker (use SelectFolderDialog, not a control):
var dlg = new SelectFolderDialog { Title = "Select folder" };
if (dlg.ShowDialog(parentControl) == DialogResult.Ok)
    string path = dlg.Directory;
```

> **Pitfall:** `TextArea` does NOT have `PlaceholderText` — only `TextBox` does. Using it on TextArea causes a compile error.

## DynamicLayout

```csharp
var layout = new DynamicLayout 
{ 
    Padding = new Padding(10), 
    Spacing = new Size(5, 5),
    DefaultSpacing = new Size(5, 5)
};

// AddRow: items fill available width proportionally
layout.AddRow(new Label { Text = "Name:" }, new TextBox());

// AddSeparateRow: creates a new row group (items don't stretch)
layout.AddSeparateRow(null, okButton, cancelButton); // null = spacer

// Scaled row: control fills remaining height
layout.Add(new TextArea(), yscale: true); // vertically scalable

// Begin/End row for complex rows
layout.BeginHorizontal();
layout.Add(labelA);
layout.Add(textBoxA);
layout.Add(new Button { Text = "..." });
layout.EndHorizontal();
```

## TableLayout

```csharp
var table = new TableLayout(columns: 2, rows: 3)
{
    Padding = new Padding(5),
    Spacing = new Size(5, 5)
};
table.Add(new Label { Text = "X:" }, 0, 0);
table.Add(xTextBox, 1, 0);
table.Add(new Label { Text = "Y:" }, 0, 1);
table.Add(yTextBox, 1, 1);
table.Add(new Label { Text = "Z:" }, 0, 2);
table.Add(zTextBox, 1, 2);
```

## UseRhinoStyle

Always call `this.UseRhinoStyle()` on dialogs and forms to match Rhino's theming:

```csharp
// Extension method from Rhino.UI
myDialog.UseRhinoStyle();
myPanel.UseRhinoStyle(); // also works on Panel subclasses
```

This handles:
- Dark/light mode switching
- Font and DPI scaling
- Rhino-native color tokens

## Dockable Side Panel (IPanel)

```csharp
// 1. Implement IPanel — class MUST have [Guid] attribute
[System.Runtime.InteropServices.Guid("YOUR-RANDOMLY-GENERATED-GUID")]
public class MyPanel : Panel, IPanel
{
    // PanelId must match the [Guid] attribute above exactly
    public static readonly Guid PanelId = new Guid("YOUR-RANDOMLY-GENERATED-GUID");

    public MyPanel(uint documentSerialNumber)
    {
        _docSerialNumber = documentSerialNumber;
        BuildUI();
    }

    private uint _docSerialNumber;

    private void BuildUI()
    {
        var layout = new DynamicLayout { Padding = new Padding(5) };
        layout.AddRow(new Label { Text = "Panel Content" });
        Content = layout;
        this.UseRhinoStyle();
    }

    void IPanel.PanelShown(uint documentSerialNumber, ShowPanelReason reason) { }
    void IPanel.PanelHidden(uint documentSerialNumber, ShowPanelReason reason) { }

    void IPanel.PanelClosing(uint documentSerialNumber, bool onCloseDocument)
    {
        // ALWAYS cancel async operations here — even if onCloseDocument is false.
        // If a background async task (streaming, HTTP polling) keeps calling
        // Application.Instance.AsyncInvoke() after the UI is destroyed,
        // Rhino exits uncleanly → .rhl lock files are not deleted →
        // .3dm files open as read-only on next launch.
        _myView?.CancelPendingWork();   // cancel CancellationTokenSource in sub-views

        if (onCloseDocument)
        {
            _myService?.Dispose();      // kill Python processes, release resources
        }
    }
}

// 2. Register in plugin's OnLoad — use 4-parameter overload
protected override LoadReturnCode OnLoad(ref string errorMessage)
{
    Rhino.UI.Panels.RegisterPanel(this, typeof(MyPanel), "My Panel", null);
    return LoadReturnCode.Success;
}

// 3. Show the panel programmatically
Rhino.UI.Panels.OpenPanel(MyPanel.PanelId);
Rhino.UI.Panels.ClosePanel(MyPanel.PanelId);
bool isOpen = Rhino.UI.Panels.IsPanelVisible(MyPanel.PanelId);
```

> **Common errors:**
> - Missing `[Guid]` on panel class → `type must have a GuidAttribute`
> - `PanelId` doesn't match `[Guid]` → panel won't open correctly
> - `PanelType.PerDocument` 5-param overload may not exist — use 4-param overload

## Inline Image Display in Chat/List Views

`TextArea` cannot embed images. To display images inline with text (e.g., a chat log with thumbnails), use a `StackLayout` of per-message panels:

```csharp
private StackLayout _messageStack = new StackLayout
{
    Orientation = Orientation.Vertical,
    Spacing = 2,
    Padding = new Padding(4),
    HorizontalContentAlignment = HorizontalAlignment.Stretch
};

// Wrap in Scrollable
var scrollable = new Scrollable { Content = _messageStack, ExpandContentWidth = true };

// Add a message with optional inline image
private Label AddMessageBubble(string sender, string text, string imageBase64, bool isUser)
{
    var headerLabel = new Label
    {
        Text = isUser ? "── You ──" : $"── {sender} ──",
        TextColor = isUser ? Colors.SkyBlue : Colors.LightGreen,
        Font = new Font(SystemFont.Bold, 9)
    };

    var contentLabel = new Label { Text = text, Wrap = WrapMode.Word };

    var bubble = new DynamicLayout { Spacing = new Size(0, 2) };
    bubble.AddRow(headerLabel);

    if (!string.IsNullOrEmpty(imageBase64))
    {
        try
        {
            var bytes = Convert.FromBase64String(imageBase64);
            using (var ms = new MemoryStream(bytes))
            {
                var img = new ImageView { Image = new Bitmap(ms), Size = new Size(240, 135) };
                bubble.AddRow(img);
            }
        }
        catch { }
    }

    bubble.AddRow(contentLabel);

    Application.Instance.AsyncInvoke(() =>
    {
        _messageStack.Items.Add(new StackLayoutItem(bubble, HorizontalAlignment.Stretch));
        ScrollToBottom();
    });

    return contentLabel; // return for streaming text updates
}

private void ScrollToBottom()
{
    Application.Instance.AsyncInvoke(() =>
    {
        _scrollable.ScrollPosition = new Point(0, int.MaxValue);
    });
}
```

## Thread Safety

Rhino UI must run on the UI thread. From background threads or async methods:

```csharp
// Marshal to UI thread (Eto)
Application.Instance.AsyncInvoke(() =>
{
    myLabel.Text = "Updated from background thread";
});

// Or with RhinoApp
RhinoApp.InvokeOnUiThread(new Action(() =>
{
    myButton.Enabled = true;
}));
```

## Event Handling — Avoid Handler Accumulation

**Anti-pattern:** Subscribing a new lambda every time a method is called. Handlers accumulate and fire multiple times.

```csharp
// BAD — if this method is called multiple times, handlers stack up:
private void SetupButton()
{
    _sendButton.Click += (s, e) => DoSomething(); // accumulates on each call!
}
```

**Correct pattern:** Subscribe once with a named method:

```csharp
// GOOD — subscribe once in BuildUI, never again:
_sendButton.Click += OnSendButtonClick;

private void OnSendButtonClick(object sender, EventArgs e)
{
    if (_busy)
        _cts?.Cancel();   // toggle: stop if running
    else
        _ = StartAsync(); // toggle: start if idle
}
```

Other event examples:
```csharp
// TextBox changes
textBox.TextChanged += (sender, e) => {
    bool valid = double.TryParse(textBox.Text, out _);
    textBox.BackgroundColor = valid ? Colors.Transparent : Colors.LightSalmon;
};

// DropDown selection changed
dropDown.SelectedIndexChanged += (sender, e) => {
    string selected = (dropDown.SelectedValue as ListItem)?.Text;
};

// Keyboard shortcut in TextArea (Ctrl+Enter to submit)
_inputArea.KeyDown += (sender, e) => {
    if (e.Key == Keys.Enter && e.Modifiers == Keys.Control)
    {
        e.Handled = true;
        _ = SendAsync();
    }
};
```

## Async Cleanup — Prevent Read-Only .3dm Files

**Symptom:** After using the plugin, Rhino `.3dm` files can only be opened as read-only on the next launch.

**Cause:** Rhino creates a `.rhl` lock file when it opens a `.3dm` file. The lock file is deleted on clean exit. If Rhino exits uncleanly (e.g., due to a crash or unhandled exception), the lock file remains and the next open is flagged as read-only.

**Root cause in plugins:** A background async task (streaming AI response, HTTP polling) keeps calling `Application.Instance.AsyncInvoke()` after the UI controls have been destroyed. Writing to a destroyed Label/Panel causes an unhandled exception → Rhino exits uncleanly.

**Fix pattern — cancel all async work in `PanelClosing`:**

```csharp
// In a sub-view that does async work (streaming, polling, HTTP):
private volatile bool _panelClosed;

public void CancelPendingWork()
{
    _panelClosed = true;        // set flag BEFORE cancelling CTS
    try { _cts?.Cancel(); } catch { }
}

// Guard EVERY AsyncInvoke call — both before enqueue and inside the lambda:
private void UpdateLabel(string text)
{
    if (_panelClosed) return;
    Application.Instance.AsyncInvoke(() =>
    {
        if (_panelClosed) return;   // may have been queued before close
        _myLabel.Text = text;
    });
}

// In PanelClosing — cancel BEFORE the UI is torn down:
void IPanel.PanelClosing(uint documentSerialNumber, bool onCloseDocument)
{
    // Cancel async operations unconditionally (not just when onCloseDocument).
    // Panel close also destroys UI controls — any pending AsyncInvoke would crash.
    _myView?.CancelPendingWork();

    if (onCloseDocument)
        _myService?.Dispose();
}
```

**Critical: cancelling the CTS alone is NOT enough.** The `finally` / `catch` blocks of the cancelled task still run and call `AsyncInvoke`. Setting `_panelClosed = true` FIRST ensures those queued lambdas become no-ops when they fire.

**Fix pattern — polling loops must check the flag:**

```csharp
// BAD: infinite loop with no exit on panel close
_ = Task.Run(async () =>
{
    while (true)
    {
        await Task.Delay(2000);
        var s = await GetStatusAsync();
        Application.Instance.AsyncInvoke(() => UpdateUI(s));  // crashes after close!
        if (!s.Indexing) break;
    }
});

// GOOD: exit when panel closes, guard every AsyncInvoke
_ = Task.Run(async () =>
{
    while (!_panelClosed)
    {
        await Task.Delay(2000);
        if (_panelClosed) break;
        var s = await GetStatusAsync();
        if (_panelClosed) break;
        Application.Instance.AsyncInvoke(() =>
        {
            if (_panelClosed) return;
            UpdateUI(s);
        });
        if (!s.Indexing) break;
    }
});
```

**Fix pattern — background startup tasks must also be cancellable:**

```csharp
// BAD: CancellationToken.None — cannot be stopped, hangs Rhino shutdown
_ = Task.Run(async () => await StartServiceAsync(CancellationToken.None));

// GOOD: use a CTS that is cancelled in Dispose()
private readonly CancellationTokenSource _startupCts = new CancellationTokenSource();

// In constructor:
_ = Task.Run(async () => {
    try { await StartServiceAsync(_startupCts.Token); }
    catch { }
});

// In Dispose():
try { _startupCts.Cancel(); _startupCts.Dispose(); } catch { }
```

**To recover existing locked files:** delete the `.rhl` file next to the `.3dm` file, or use Rhino's File → Save As to overwrite and reset the lock.

## Platform Notes

```csharp
// Check platform if needed
bool isWindows = Eto.Platform.IsWinForms || Eto.Platform.IsWpf;
bool isMac = Eto.Platform.IsMac;

// Minimum dialog size for Mac (dialogs auto-size on Mac)
if (isMac) MinimumSize = new Size(300, 0);
```
