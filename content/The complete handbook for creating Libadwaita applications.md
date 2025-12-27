+++
title = "The Complete Handbook for Creating Libadwaita Applications"
date = 2025-12-27
+++

## A Comprehensive Guide for GNOME Development



## Table of Contents

1. Introduction to Libadwaita
2. Setting Up Your Development Environment on Arch Linux
3. GNOME Human Interface Guidelines (HIG)
4. Project Structure and Standards
5. Basic Application Framework
6. Simple Widgets
7. Complex Widgets
8. Layouts and Containers
9. Application Patterns
10. System Integration
11. Data Management
12. Advanced Features
13. Packaging and Distribution



## Chapter 1: Introduction to Libadwaita

### What is Libadwaita?

Libadwaita is GNOME's official library for building modern, consistent applications that integrate seamlessly with the GNOME desktop environment. It provides:

- **Adaptive widgets** that work across different form factors (desktop, tablet, mobile)
- **Modern design patterns** following GNOME's design philosophy
- **Consistent styling** that matches the GNOME Shell and other core applications
- **Animation support** for smooth, polished user experiences
- **Dark mode support** built-in and automatic
- **Accessibility features** by default

### Why Libadwaita?

Traditional GTK4 applications can look dated or inconsistent with modern GNOME. Libadwaita solves this by:

1. **Enforcing consistency**: Applications automatically match the system theme
2. **Simplifying development**: Common patterns are pre-built
3. **Future-proofing**: Your app will evolve with GNOME's design language
4. **Improving accessibility**: Built-in support for screen readers and keyboard navigation

### Language Choices: Rust vs Python

**Rust** offers:
- Memory safety without garbage collection
- Excellent performance
- Strong type system
- Growing ecosystem with gtk-rs

**Python** offers:
- Rapid prototyping
- Easier learning curve
- Immediate execution (no compilation)
- Extensive libraries for data processing

Both languages have full Libadwaita bindings and are officially supported by GNOME.



## Chapter 2: Setting Up Your Development Environment on Arch Linux

### Installing Dependencies

Arch Linux provides excellent GNOME development tools through its repositories.

#### For Rust Development:

```bash
# Install base development tools
sudo pacman -S base-devel git

# Install Rust
sudo pacman -S rust rust-analyzer

# Install GTK4 and Libadwaita
sudo pacman -S gtk4 libadwaita

# Install development tools
sudo pacman -S gnome-builder devhelp d-spy blueprint-compiler

# Optional: Install Flatpak for testing
sudo pacman -S flatpak flatpak-builder
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install flathub org.gnome.Sdk//47 org.gnome.Platform//47
```

#### For Python Development:

```bash
# Install Python and GTK bindings
sudo pacman -S python python-gobject gtk4 libadwaita

# Install development tools
sudo pacman -S gnome-builder python-pip

# Install additional Python tools
pip install --user pygobject-stubs
```

### Development Tools Overview

**GNOME Builder**: The official IDE for GNOME development
- Integrated project templates
- Built-in GTK Inspector
- Automatic code completion
- Direct Flatpak support

**DevHelp**: Documentation browser for GTK, Libadwaita, and other libraries

**D-Spy**: D-Bus debugger for testing system integration

**Blueprint**: High-level markup language for UI definitions (compiles to GTK's UI format)



## Chapter 3: GNOME Human Interface Guidelines (HIG)

### Core Principles

The GNOME HIG defines how applications should look, feel, and behave. Understanding these principles is crucial for creating authentic GNOME applications.

#### 1. **Simplicity First**

GNOME applications should be focused and uncluttered:
- Avoid overwhelming users with options
- Present features progressively (basic → advanced)
- Use clear, concise language
- Hide complexity behind sensible defaults

**Example**: Instead of showing 20 export options upfront, provide a simple "Export" button with an "Advanced Options" expander.

#### 2. **Consistency**

Users should feel at home across all GNOME applications:
- Use standard widgets for common actions
- Follow established patterns (e.g., HeaderBar layout)
- Maintain consistent terminology
- Use the same keyboard shortcuts as other apps

#### 3. **Respect User Agency**

Never surprise or mislead users:
- Clearly communicate what actions will do
- Make destructive actions obvious and reversible
- Save work automatically when possible
- Avoid modal dialogs except for critical decisions

#### 4. **Accessibility by Default**

Every user should be able to use your application:
- Proper focus management
- Keyboard navigation for all functions
- Screen reader support via proper labels
- Sufficient color contrast
- Clear, resizable text

### Visual Design Standards

#### Typography

Libadwaita uses the system font with specific size classes:

- **Display**: Large, prominent text (32pt+)
- **Title 1-4**: Hierarchical headings (20pt → 15pt)
- **Heading**: Section headers (15pt, bold)
- **Body**: Regular text (11pt)
- **Caption**: Secondary information (9pt)

#### Spacing

GNOME uses a consistent 6-pixel spacing grid:

- **6px**: Minimum spacing between related elements
- **12px**: Standard spacing between widgets
- **18px**: Spacing between groups
- **24px**: Large spacing between sections
- **32px**: Maximum spacing for emphasis

#### Color Semantics

Colors have specific meanings in GNOME:

- **Blue (Accent)**: Primary actions, selection
- **Green (Success)**: Completed actions, positive status
- **Yellow (Warning)**: Caution, non-critical issues
- **Red (Error)**: Errors, destructive actions
- **Purple**: In-progress operations

These colors automatically adapt to dark mode.

### Application Structure Patterns

#### The HeaderBar

Every GNOME application should use an `AdwHeaderBar`:

```
┌─────────────────────────────────────────┐
│ [←] Application Name         [≡] [Menu] │ HeaderBar
├─────────────────────────────────────────┤
│                                         │
│                                         │
│          Content Area                   │
│                                         │
│                                         │
└─────────────────────────────────────────┘
```

**Rules**:
- Title in the center (hidden on narrow screens)
- Navigation on the left (back button, split view button)
- Primary actions on the right
- Menu button always rightmost

#### Navigation Patterns

**1. Flat Navigation** (single view):
```
HeaderBar with title
→ Content
```

**2. Back Navigation** (hierarchical):
```
HeaderBar with back button
→ Stack/Navigation View
```

**3. Sidebar Navigation** (multiple sections):
```
AdwNavigationSplitView
├─ Sidebar (categories)
└─ Content
```

**4. Tabbed Navigation** (parallel sections):
```
AdwTabBar or AdwViewSwitcher
→ Multiple views
```

### Content Organization

#### Preference Windows

Use `AdwPreferencesWindow` for settings:

```
┌─────────────────────────────────────────┐
│ [×] Preferences                         │
├─────────┬───────────────────────────────┤
│ General │ ┌───────────────────────────┐ │
│ Privacy │ │ Section Title              │ │
│ About   │ │                           │ │
│         │ │ ○ Radio Option 1          │ │
│         │ │ ○ Radio Option 2          │ │
│         │ │                           │ │
│         │ │ Another Section           │ │
│         │ │ [Toggle Switch]    ─┐    │ │
│         │ │                           │ │
│         │ └───────────────────────────┘ │
└─────────┴───────────────────────────────┘
```

Features:
- Sidebar for categories (auto-hides on mobile)
- Grouped preferences with clear sections
- Descriptive subtitles for complex options
- Search functionality (automatic)

#### Lists and Content

Use `AdwBin` and proper list patterns:

**Do**:
- Use `GtkListView` for large, dynamic lists
- Use `AdwPreferencesGroup` for settings-style lists
- Provide search/filter for long lists
- Show empty states with helpful messages

**Don't**:
- Use `GtkTreeView` (deprecated pattern)
- Show empty white screens
- Mix different list styles in one view



## Chapter 4: Project Structure and Standards

### Rust Project Structure

A typical Rust Libadwaita application follows this structure:

```
my-app/
├── Cargo.toml
├── data/
│   ├── resources.gresource.xml
│   ├── com.example.MyApp.desktop.in
│   ├── com.example.MyApp.metainfo.xml.in
│   ├── com.example.MyApp.gschema.xml
│   └── icons/
│       └── com.example.MyApp.svg
├── src/
│   ├── main.rs
│   ├── application.rs
│   ├── window.rs
│   ├── widgets/
│   │   ├── mod.rs
│   │   └── custom_widget.rs
│   └── config.rs
├── po/
│   ├── POTFILES.in
│   └── LINGUAS
└── build.rs
```

#### Cargo.toml Template

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]

[dependencies]
gtk = { version = "0.9", package = "gtk4", features = ["v4_12"] }
adwaita = { version = "0.7", package = "libadwaita", features = ["v1_6"] }
glib = "0.20"
gio = "0.20"
log = "0.4"
env_logger = "0.11"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

[build-dependencies]
glib-build-tools = "0.20"

[profile.release]
lto = true
codegen-units = 1
opt-level = 3
strip = true
```

### Python Project Structure

A Python Libadwaita application structure:

```
my-app/
├── setup.py
├── data/
│   ├── com.example.MyApp.desktop.in
│   ├── com.example.MyApp.metainfo.xml.in
│   ├── com.example.MyApp.gschema.xml
│   ├── resources.gresource.xml
│   └── ui/
│       └── window.ui
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── application.py
│   ├── window.py
│   └── widgets/
│       ├── __init__.py
│       └── custom_widget.py
├── po/
│   ├── POTFILES.in
│   └── LINGUAS
└── meson.build
```

### Application Naming Standards

GNOME applications follow reverse-DNS naming:

**Application ID**: `com.example.MyApp`
- Must be unique
- Use lowercase with capitals for app name
- Match across all files (desktop, metainfo, GSchema)

**File naming**:
- Desktop file: `com.example.MyApp.desktop`
- Metainfo: `com.example.MyApp.metainfo.xml`
- GSettings schema: `com.example.MyApp.gschema.xml`
- Icon: `com.example.MyApp.svg` (or .png)
- D-Bus name: `com.example.MyApp`

### Build System: Meson

GNOME projects use Meson for building. Example `meson.build`:

```meson
project('my-app',
  version: '0.1.0',
  meson_version: '>= 0.59.0',
)

i18n = import('i18n')
gnome = import('gnome')

# Configuration
conf = configuration_data()
conf.set_quoted('APP_ID', 'com.example.MyApp')
conf.set_quoted('VERSION', meson.project_version())
conf.set_quoted('GETTEXT_PACKAGE', 'my-app')
conf.set_quoted('LOCALEDIR', get_option('prefix') / get_option('localedir'))

# Subdirectories
subdir('data')
subdir('src')
subdir('po')

# Post-install scripts
gnome.post_install(
  gtk_update_icon_cache: true,
  glib_compile_schemas: true,
  update_desktop_database: true,
)
```



## Chapter 5: Basic Application Framework

### Minimal Rust Application

Let's build a complete minimal application in Rust:

#### main.rs

```rust
mod application;
mod config;
mod window;

use application::MyApplication;
use config::{APP_ID, PKGDATADIR, PROFILE, VERSION};
use gettextrs::{gettext, LocaleCategory};
use gtk::prelude::*;
use gtk::{gio, glib};

fn main() -> glib::ExitCode {
    // Initialize logger
    env_logger::init();
    
    // Initialize translations
    gettextrs::setlocale(LocaleCategory::LcAll, "");
    gettextrs::bindtextdomain("my-app", LOCALEDIR).expect("Failed to bind text domain");
    gettextrs::textdomain("my-app").expect("Failed to set text domain");

    // Load resources
    let resources = gio::Resource::load(PKGDATADIR.to_owned() + "/my-app.gresource")
        .expect("Could not load resources");
    gio::resources_register(&resources);

    // Create application
    let app = MyApplication::new(APP_ID, &gio::ApplicationFlags::default());
    
    app.run()
}
```

#### config.rs

```rust
pub const APP_ID: &str = "com.example.MyApp";
pub const VERSION: &str = env!("CARGO_PKG_VERSION");
pub const PKGDATADIR: &str = "/usr/share/my-app";
pub const LOCALEDIR: &str = "/usr/share/locale";
pub const PROFILE: &str = "development"; // or "release"
```

#### application.rs

```rust
use adwaita::prelude::*;
use adwaita::subclass::prelude::*;
use gtk::{gio, glib};

use crate::config::VERSION;
use crate::window::MyApplicationWindow;

mod imp {
    use super::*;

    #[derive(Debug, Default)]
    pub struct MyApplication {}

    #[glib::object_subclass]
    impl ObjectSubclass for MyApplication {
        const NAME: &'static str = "MyApplication";
        type Type = super::MyApplication;
        type ParentType = adwaita::Application;
    }

    impl ObjectImpl for MyApplication {
        fn constructed(&self) {
            self.parent_constructed();
            let obj = self.obj();
            obj.setup_gactions();
            obj.set_accels_for_action("app.quit", &["<primary>q"]);
            obj.set_accels_for_action("window.close", &["<primary>w"]);
        }
    }

    impl ApplicationImpl for MyApplication {
        fn activate(&self) {
            let application = self.obj();
            
            // Get or create window
            let window = if let Some(window) = application.active_window() {
                window
            } else {
                let window = MyApplicationWindow::new(&*application);
                window.upcast()
            };

            window.present();
        }
    }

    impl GtkApplicationImpl for MyApplication {}
    impl AdwApplicationImpl for MyApplication {}
}

glib::wrapper! {
    pub struct MyApplication(ObjectSubclass<imp::MyApplication>)
        @extends gio::Application, gtk::Application, adwaita::Application,
        @implements gio::ActionGroup, gio::ActionMap;
}

impl MyApplication {
    pub fn new(application_id: &str, flags: &gio::ApplicationFlags) -> Self {
        glib::Object::builder()
            .property("application-id", application_id)
            .property("flags", flags)
            .build()
    }

    fn setup_gactions(&self) {
        // Quit action
        let quit_action = gio::SimpleAction::new("quit", None);
        quit_action.connect_activate(glib::clone!(
            #[weak(rename_to = app)]
            self,
            move |_, _| app.quit()
        ));
        self.add_action(&quit_action);

        // About action
        let about_action = gio::SimpleAction::new("about", None);
        about_action.connect_activate(glib::clone!(
            #[weak(rename_to = app)]
            self,
            move |_, _| app.show_about()
        ));
        self.add_action(&about_action);
    }

    fn show_about(&self) {
        let window = self.active_window().unwrap();
        let about = adwaita::AboutDialog::builder()
            .application_name("My Application")
            .application_icon("com.example.MyApp")
            .developer_name("Your Name")
            .version(VERSION)
            .developers(vec!["Your Name <your.email@example.com>"])
            .copyright("© 2025 Your Name")
            .license_type(gtk::License::Gpl30)
            .website("https://example.com")
            .issue_url("https://github.com/example/my-app/issues")
            .build();

        about.present(Some(&window));
    }
}
```

#### window.rs

```rust
use adwaita::prelude::*;
use adwaita::subclass::prelude::*;
use gtk::{gio, glib};

mod imp {
    use super::*;

    #[derive(Debug, Default, gtk::CompositeTemplate)]
    #[template(resource = "/com/example/MyApp/window.ui")]
    pub struct MyApplicationWindow {
        #[template_child]
        pub label: TemplateChild<gtk::Label>,
    }

    #[glib::object_subclass]
    impl ObjectSubclass for MyApplicationWindow {
        const NAME: &'static str = "MyApplicationWindow";
        type Type = super::MyApplicationWindow;
        type ParentType = adwaita::ApplicationWindow;

        fn class_init(klass: &mut Self::Class) {
            klass.bind_template();
        }

        fn instance_init(obj: &glib::subclass::InitializingObject<Self>) {
            obj.init_template();
        }
    }

    impl ObjectImpl for MyApplicationWindow {
        fn constructed(&self) {
            self.parent_constructed();
            
            // Load window state
            let obj = self.obj();
            obj.load_window_size();
        }
    }

    impl WidgetImpl for MyApplicationWindow {}
    impl WindowImpl for MyApplicationWindow {
        fn close_request(&self) -> glib::Propagation {
            // Save window state
            self.obj().save_window_size();
            self.parent_close_request()
        }
    }
    impl ApplicationWindowImpl for MyApplicationWindow {}
    impl AdwApplicationWindowImpl for MyApplicationWindow {}
}

glib::wrapper! {
    pub struct MyApplicationWindow(ObjectSubclass<imp::MyApplicationWindow>)
        @extends gtk::Widget, gtk::Window, gtk::ApplicationWindow, adwaita::ApplicationWindow,
        @implements gio::ActionGroup, gio::ActionMap;
}

impl MyApplicationWindow {
    pub fn new(app: &adwaita::Application) -> Self {
        glib::Object::builder()
            .property("application", app)
            .build()
    }

    fn save_window_size(&self) {
        let settings = gio::Settings::new("com.example.MyApp");
        let size = self.default_size();
        
        settings.set_int("window-width", size.0).ok();
        settings.set_int("window-height", size.1).ok();
        settings.set_boolean("is-maximized", self.is_maximized()).ok();
    }

    fn load_window_size(&self) {
        let settings = gio::Settings::new("com.example.MyApp");
        
        let width = settings.int("window-width");
        let height = settings.int("window-height");
        let is_maximized = settings.boolean("is-maximized");

        self.set_default_size(width, height);
        
        if is_maximized {
            self.maximize();
        }
    }
}
```

#### window.ui (GTK UI Definition)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<interface>
  <template class="MyApplicationWindow" parent="AdwApplicationWindow">
    <property name="default-width">600</property>
    <property name="default-height">400</property>
    <property name="content">
      <object class="AdwToolbarView">
        <child type="top">
          <object class="AdwHeaderBar">
            <child type="end">
              <object class="GtkMenuButton">
                <property name="icon-name">open-menu-symbolic</property>
                <property name="menu-model">primary_menu</property>
                <property name="tooltip-text" translatable="yes">Main Menu</property>
              </object>
            </child>
          </object>
        </child>
        <property name="content">
          <object class="AdwStatusPage">
            <property name="icon-name">com.example.MyApp</property>
            <property name="title" translatable="yes">Welcome</property>
            <property name="description" translatable="yes">This is your first Libadwaita application</property>
            <property name="child">
              <object class="GtkLabel" id="label">
                <property name="label" translatable="yes">Hello, GNOME!</property>
                <style>
                  <class name="title-1"/>
                </style>
              </object>
            </property>
          </object>
        </property>
      </object>
    </property>
  </template>
  
  <menu id="primary_menu">
    <section>
      <item>
        <attribute name="label" translatable="yes">_About My App</attribute>
        <attribute name="action">app.about</attribute>
      </item>
    </section>
  </menu>
</interface>
```

### Minimal Python Application

Now the same application in Python:

#### main.py

```python
#!/usr/bin/env python3

import sys
import gi

gi.require_version('Gtk', '4.0')
gi.require_version('Adw', '1')

from gi.repository import Gtk, Adw, Gio

from .application import MyApplication

def main():
    """The application's entry point."""
    app = MyApplication()
    return app.run(sys.argv)
```

#### application.py

```python
import gi

gi.require_version('Gtk', '4.0')
gi.require_version('Adw', '1')

from gi.repository import Gtk, Adw, Gio

from .window import MyApplicationWindow

class MyApplication(Adw.Application):
    """The main application class."""

    def __init__(self):
        super().__init__(
            application_id='com.example.MyApp',
            flags=Gio.ApplicationFlags.DEFAULT_FLAGS
        )
        
        self.create_action('quit', lambda *_: self.quit(), ['<primary>q'])
        self.create_action('about', self.on_about_action)

    def do_activate(self):
        """Called when the application is activated."""
        win = self.props.active_window
        if not win:
            win = MyApplicationWindow(application=self)
        win.present()

    def on_about_action(self, widget, _):
        """Callback for the app.about action."""
        about = Adw.AboutDialog(
            application_name='My Application',
            application_icon='com.example.MyApp',
            developer_name='Your Name',
            version='0.1.0',
            developers=['Your Name <your.email@example.com>'],
            copyright='© 2025 Your Name',
            license_type=Gtk.License.GPL_3_0,
            website='https://example.com',
            issue_url='https://github.com/example/my-app/issues',
        )
        about.present(self.props.active_window)

    def create_action(self, name, callback, shortcuts=None):
        """Add an application action.
        
        Args:
            name: the name of the action
            callback: the function to be called when the action is activated
            shortcuts: an optional list of accelerators
        """
        action = Gio.SimpleAction.new(name, None)
        action.connect("activate", callback)
        self.add_action(action)
        if shortcuts:
            self.set_accels_for_action(f"app.{name}", shortcuts)
```

#### window.py

```python
import gi

gi.require_version('Gtk', '4.0')
gi.require_version('Adw', '1')

from gi.repository import Gtk, Adw, Gio

@Gtk.Template(resource_path='/com/example/MyApp/window.ui')
class MyApplicationWindow(Adw.ApplicationWindow):
    __gtype_name__ = 'MyApplicationWindow'

    label = Gtk.Template.Child()

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        
        # Load window state
        self.settings = Gio.Settings.new('com.example.MyApp')
        self.load_window_state()

    def load_window_state(self):
        """Load the window state from GSettings."""
        width = self.settings.get_int('window-width')
        height = self.settings.get_int('window-height')
        is_maximized = self.settings.get_boolean('is-maximized')
        
        self.set_default_size(width, height)
        
        if is_maximized:
            self.maximize()

    def save_window_state(self):
        """Save the window state to GSettings."""
        size = self.get_default_size()
        self.settings.set_int('window-width', size.width)
        self.settings.set_int('window-height', size.height)
        self.settings.set_boolean('is-maximized', self.is_maximized())

    def do_close_request(self):
        """Called when the window is closed."""
        self.save_window_state()
        return False
```

The UI file (window.ui) is the same for both languages.

### GSettings Schema

Both applications need a GSettings schema for storing preferences:

#### com.example.MyApp.gschema.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<schemalist>
  <schema id="com.example.MyApp" path="/com/example/MyApp/">
    <key name="window-width" type="i">
      <default>600</default>
      <summary>Window width</summary>
      <description>The width of the main window</description>
    </key>
    <key name="window-height" type="i">
      <default>400</default>
      <summary>Window height</summary>
      <description>The height of the main window</description>
    </key>
    <key name="is-maximized" type="b">
      <default>false</default>
      <summary>Window maximized state</summary>
      <description>Whether the window is maximized</description>
    </key>
  </schema>
</schemalist>
```

After installation, compile the schema:
```bash
sudo glib-compile-schemas /usr/share/glib-2.0/schemas/
```



## Chapter 6: Simple Widgets

### Understanding Widget Basics

Libadwaita widgets inherit from GTK4 widgets and add:
- Automatic dark mode support
- Rounded corners and modern styling
- Animation capabilities
- Responsive behavior

### AdwStatusPage

The status page is used for empty states, welcome screens, and error messages.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_status_page() -> adwaita::StatusPage {
    adwaita::StatusPage::builder()
        .icon_name("folder-symbolic")
        .title("No Files")
        .description("Drop files here to get started")
        .build()
}

// With a child widget
fn create_status_page_with_button() -> adwaita::StatusPage {
    let button = gtk::Button::with_label("Open Folder");
    button.add_css_class("pill");
    button.add_css_class("suggested-action");
    
    adwaita::StatusPage::builder()
        .icon_name("folder-open-symbolic")
        .title("Welcome")
        .description("Choose a folder to begin")
        .child(&button)
        .build()
}
```

**Python Example**:

```python
def create_status_page():
    """Create a status page for empty state."""
    status_page = Adw.StatusPage(
        icon_name='folder-symbolic',
        title='No Files',
        description='Drop files here to get started'
    )
    return status_page

def create_status_page_with_button():
    """Create a status page with an action button."""
    button = Gtk.Button(label='Open Folder')
    button.add_css_class('pill')
    button.add_css_class('suggested-action')
    
    status_page = Adw.StatusPage(
        icon_name='folder-open-symbolic',
        title='Welcome',
        description='Choose a folder to begin',
        child=button
    )
    return status_page
```

**CSS Classes**:
- `pill`: Rounded button shape
- `suggested-action`: Blue accent (primary action)
- `destructive-action`: Red color (dangerous action)
- `flat`: Remove button background

### AdwToastOverlay and AdwToast

Toasts provide non-intrusive feedback to users.

**Rust Example**:

```rust
use adwaita::prelude::*;

// Setup (in window initialization)
let toast_overlay = adwaita::ToastOverlay::new();
let content = gtk::Box::new(gtk::Orientation::Vertical, 0);
toast_overlay.set_child(Some(&content));

// Show a toast
fn show_toast(overlay: &adwaita::ToastOverlay, message: &str) {
    let toast = adwaita::Toast::new(message);
    toast.set_timeout(3); // 3 seconds
    overlay.add_toast(toast);
}

// Toast with action
fn show_toast_with_action(overlay: &adwaita::ToastOverlay) {
    let toast = adwaita::Toast::new("File deleted");
    toast.set_button_label(Some("Undo"));
    toast.set_action_name(Some("app.undo-delete"));
    overlay.add_toast(toast);
}

// Priority toast (shown immediately)
fn show_priority_toast(overlay: &adwaita::ToastOverlay) {
    let toast = adwaita::Toast::new("Connection lost");
    toast.set_priority(adwaita::ToastPriority::High);
    overlay.add_toast(toast);
}
```

**Python Example**:

```python
def setup_toast_overlay(self):
    """Setup toast overlay in window."""
    self.toast_overlay = Adw.ToastOverlay()
    content = Gtk.Box(orientation=Gtk.Orientation.VERTICAL)
    self.toast_overlay.set_child(content)
    
def show_toast(self, message):
    """Show a simple toast message."""
    toast = Adw.Toast.new(message)
    toast.set_timeout(3)  # 3 seconds
    self.toast_overlay.add_toast(toast)

def show_toast_with_action(self):
    """Show a toast with an action button."""
    toast = Adw.Toast.new("File deleted")
    toast.set_button_label("Undo")
    toast.set_action_name("app.undo-delete")
    self.toast_overlay.add_toast(toast)

def show_priority_toast(self):
    """Show a high-priority toast."""
    toast = Adw.Toast.new("Connection lost")
    toast.set_priority(Adw.ToastPriority.HIGH)
    self.toast_overlay.add_toast(toast)
```

### AdwBanner

Banners display important, persistent messages at the top of content.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_banner() -> adwaita::Banner {
    let banner = adwaita::Banner::new("Your trial expires in 3 days");
    banner.set_button_label(Some("Upgrade"));
    banner.connect_button_clicked(|_| {
        println!("Upgrade clicked");
    });
    banner
}

// Info banner (blue)
fn create_info_banner() -> adwaita::Banner {
    adwaita::Banner::builder()
        .title("A new version is available")
        .button_label("Download")
        .build()
}

// Warning banner (yellow)
fn create_warning_banner() -> adwaita::Banner {
    let banner = adwaita::Banner::new("Limited connectivity");
    banner.add_css_class("warning");
    banner
}

// Show/hide banner programmatically
fn toggle_banner(banner: &adwaita::Banner, show: bool) {
    banner.set_revealed(show);
}
```

**Python Example**:

```python
def create_banner(self):
    """Create a banner with action."""
    banner = Adw.Banner(title="Your trial expires in 3 days")
    banner.set_button_label("Upgrade")
    banner.connect('button-clicked', self.on_upgrade_clicked)
    return banner

def create_info_banner(self):
    """Create an info banner."""
    return Adw.Banner(
        title="A new version is available",
        button_label="Download"
    )

def create_warning_banner(self):
    """Create a warning banner."""
    banner = Adw.Banner(title="Limited connectivity")
    banner.add_css_class('warning')
    return banner

def toggle_banner(self, banner, show):
    """Show or hide banner."""
    banner.set_revealed(show)
```

### AdwActionRow

Action rows are the foundation of list-based interfaces in Libadwaita.

**Rust Example**:

```rust
use adwaita::prelude::*;

// Simple action row
fn create_simple_row() -> adwaita::ActionRow {
    adwaita::ActionRow::builder()
        .title("Bluetooth")
        .subtitle("Connected to 2 devices")
        .activatable(true)
        .build()
}

// Action row with icon
fn create_row_with_icon() -> adwaita::ActionRow {
    let row = adwaita::ActionRow::builder()
        .title("Wi-Fi")
        .subtitle("MyNetwork")
        .activatable(true)
        .build();
    
    let icon = gtk::Image::from_icon_name("network-wireless-symbolic");
    row.add_prefix(&icon);
    
    row
}

// Action row with toggle
fn create_row_with_toggle() -> adwaita::ActionRow {
    let row = adwaita::ActionRow::builder()
        .title("Night Light")
        .subtitle("Reduces blue light at night")
        .build();
    
    let toggle = gtk::Switch::new();
    toggle.set_valign(gtk::Align::Center);
    row.add_suffix(&toggle);
    row.set_activatable_widget(Some(&toggle));
    
    row
}

// Action row with disclosure indicator
fn create_expandable_row() -> adwaita::ActionRow {
    let row = adwaita::ActionRow::builder()
        .title("Network Settings")
        .subtitle("Configure network preferences")
        .activatable(true)
        .build();
    
    let chevron = gtk::Image::from_icon_name("go-next-symbolic");
    row.add_suffix(&chevron);
    
    row.connect_activated(|_| {
        println!("Navigate to network settings");
    });
    
    row
}

// Action row with badge
fn create_row_with_badge() -> adwaita::ActionRow {
    let row = adwaita::ActionRow::builder()
        .title("Messages")
        .activatable(true)
        .build();
    
    let badge = gtk::Label::new(Some("3"));
    badge.add_css_class("badge");
    badge.add_css_class("numeric");
    row.add_suffix(&badge);
    
    row
}
```

**Python Example**:

```python
def create_simple_row(self):
    """Create a simple action row."""
    return Adw.ActionRow(
        title="Bluetooth",
        subtitle="Connected to 2 devices",
        activatable=True
    )

def create_row_with_icon(self):
    """Create an action row with icon prefix."""
    row = Adw.ActionRow(
        title="Wi-Fi",
        subtitle="MyNetwork",
        activatable=True
    )
    
    icon = Gtk.Image.new_from_icon_name("network-wireless-symbolic")
    row.add_prefix(icon)
    
    return row

def create_row_with_toggle(self):
    """Create an action row with toggle switch."""
    row = Adw.ActionRow(
        title="Night Light",
        subtitle="Reduces blue light at night"
    )
    
    toggle = Gtk.Switch()
    toggle.set_valign(Gtk.Align.CENTER)
    row.add_suffix(toggle)
    row.set_activatable_widget(toggle)
    
    return row

def create_expandable_row(self):
    """Create an action row with disclosure indicator."""
    row = Adw.ActionRow(
        title="Network Settings",
        subtitle="Configure network preferences",
        activatable=True
    )
    
    chevron = Gtk.Image.new_from_icon_name("go-next-symbolic")
    row.add_suffix(chevron)
    
    row.connect('activated', self.on_network_settings_clicked)
    
    return row

def create_row_with_badge(self):
    """Create an action row with a badge."""
    row = Adw.ActionRow(
        title="Messages",
        activatable=True
    )
    
    badge = Gtk.Label(label="3")
    badge.add_css_class('badge')
    badge.add_css_class('numeric')
    row.add_suffix(badge)
    
    return row
```

### AdwExpanderRow

Expander rows reveal additional content when clicked.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_expander_row() -> adwaita::ExpanderRow {
    let expander = adwaita::ExpanderRow::builder()
        .title("Advanced Settings")
        .subtitle("Configure advanced options")
        .build();
    
    // Add child rows
    let row1 = adwaita::ActionRow::builder()
        .title("Debug Mode")
        .build();
    let toggle1 = gtk::Switch::new();
    toggle1.set_valign(gtk::Align::Center);
    row1.add_suffix(&toggle1);
    row1.set_activatable_widget(Some(&toggle1));
    
    let row2 = adwaita::ActionRow::builder()
        .title("Verbose Logging")
        .build();
    let toggle2 = gtk::Switch::new();
    toggle2.set_valign(gtk::Align::Center);
    row2.add_suffix(&toggle2);
    row2.set_activatable_widget(Some(&toggle2));
    
    expander.add_row(&row1);
    expander.add_row(&row2);
    
    expander
}

// Expander row with icon
fn create_expander_with_icon() -> adwaita::ExpanderRow {
    let expander = adwaita::ExpanderRow::builder()
        .title("Notifications")
        .subtitle("3 new notifications")
        .build();
    
    let icon = gtk::Image::from_icon_name("notification-symbolic");
    expander.add_prefix(&icon);
    
    // Add notifications
    for i in 1..=3 {
        let row = adwaita::ActionRow::builder()
            .title(&format!("Notification {}", i))
            .subtitle("2 minutes ago")
            .build();
        expander.add_row(&row);
    }
    
    expander
}

// Toggle expander programmatically
fn toggle_expander(expander: &adwaita::ExpanderRow) {
    let expanded = expander.is_expanded();
    expander.set_expanded(!expanded);
}
```

**Python Example**:

```python
def create_expander_row(self):
    """Create an expander row with child rows."""
    expander = Adw.ExpanderRow(
        title="Advanced Settings",
        subtitle="Configure advanced options"
    )
    
    # Add child rows
    row1 = Adw.ActionRow(title="Debug Mode")
    toggle1 = Gtk.Switch()
    toggle1.set_valign(Gtk.Align.CENTER)
    row1.add_suffix(toggle1)
    row1.set_activatable_widget(toggle1)
    
    row2 = Adw.ActionRow(title="Verbose Logging")
    toggle2 = Gtk.Switch()
    toggle2.set_valign(Gtk.Align.CENTER)
    row2.add_suffix(toggle2)
    row2.set_activatable_widget(toggle2)
    
    expander.add_row(row1)
    expander.add_row(row2)
    
    return expander

def create_expander_with_icon(self):
    """Create an expander row with icon and multiple children."""
    expander = Adw.ExpanderRow(
        title="Notifications",
        subtitle="3 new notifications"
    )
    
    icon = Gtk.Image.new_from_icon_name("notification-symbolic")
    expander.add_prefix(icon)
    
    # Add notifications
    for i in range(1, 4):
        row = Adw.ActionRow(
            title=f"Notification {i}",
            subtitle="2 minutes ago"
        )
        expander.add_row(row)
    
    return expander

def toggle_expander(self, expander):
    """Toggle expander state programmatically."""
    expanded = expander.get_expanded()
    expander.set_expanded(not expanded)
```

### AdwComboRow

Combo rows provide a dropdown selection within a list.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_combo_row() -> adwaita::ComboRow {
    // Create string list
    let model = gtk::StringList::new(&["Light", "Dark", "Follow System"]);
    
    let combo = adwaita::ComboRow::builder()
        .title("Theme")
        .subtitle("Choose appearance")
        .model(&model)
        .build();
    
    // Set initial selection
    combo.set_selected(2); // "Follow System"
    
    // Connect to selection change
    combo.connect_selected_notify(|combo| {
        let selected = combo.selected();
        println!("Selected: {}", selected);
    });
    
    combo
}

// Combo row with custom display
fn create_combo_row_custom() -> adwaita::ComboRow {
    use gtk::gio;
    
    #[derive(Clone)]
    struct QualityOption {
        name: String,
        value: i32,
    }
    
    // Create custom model
    let store = gio::ListStore::new::<glib::BoxedAnyObject>();
    
    let options = vec![
        QualityOption { name: "Low (480p)".to_string(), value: 480 },
        QualityOption { name: "Medium (720p)".to_string(), value: 720 },
        QualityOption { name: "High (1080p)".to_string(), value: 1080 },
    ];
    
    for option in options {
        store.append(&glib::BoxedAnyObject::new(option));
    }
    
    let combo = adwaita::ComboRow::builder()
        .title("Video Quality")
        .model(&store)
        .build();
    
    // Setup expression to display name
    let expression = gtk::PropertyExpression::new(
        glib::BoxedAnyObject::static_type(),
        None::<&gtk::Expression>,
        "name",
    );
    combo.set_expression(Some(&expression));
    
    combo
}
```

**Python Example**:

```python
def create_combo_row(self):
    """Create a combo row with string options."""
    model = Gtk.StringList.new(["Light", "Dark", "Follow System"])
    
    combo = Adw.ComboRow(
        title="Theme",
        subtitle="Choose appearance",
        model=model
    )
    
    # Set initial selection
    combo.set_selected(2)  # "Follow System"
    
    # Connect to selection change
    combo.connect('notify::selected', self.on_theme_changed)
    
    return combo

def on_theme_changed(self, combo, _pspec):
    """Handle theme selection change."""
    selected = combo.get_selected()
    print(f"Selected: {selected}")

def create_combo_row_custom(self):
    """Create a combo row with custom model."""
    from gi.repository import Gio
    
    # Create list store
    store = Gio.ListStore.new(Gtk.StringObject)
    store.append(Gtk.StringObject.new("Low (480p)"))
    store.append(Gtk.StringObject.new("Medium (720p)"))
    store.append(Gtk.StringObject.new("High (1080p)"))
    
    combo = Adw.ComboRow(
        title="Video Quality",
        model=store
    )
    
    return combo
```

### AdwEntryRow

Entry rows provide inline text editing within lists.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_entry_row() -> adwaita::EntryRow {
    adwaita::EntryRow::builder()
        .title("Username")
        .build()
}

// Entry row with validation
fn create_validated_entry_row() -> adwaita::EntryRow {
    let entry = adwaita::EntryRow::builder()
        .title("Email Address")
        .build();
    
    // Add suffix icon
    let icon = gtk::Image::from_icon_name("emblem-ok-symbolic");
    icon.set_visible(false);
    entry.add_suffix(&icon);
    
    // Validate on change
    entry.connect_changed(glib::clone!(
        #[weak]
        icon,
        move |entry| {
            let text = entry.text();
            let is_valid = text.contains('@') && text.contains('.');
            icon.set_visible(is_valid);
            
            if is_valid {
                entry.remove_css_class("error");
            } else if !text.is_empty() {
                entry.add_css_class("error");
            } else {
                entry.remove_css_class("error");
            }
        }
    ));
    
    entry
}

// Password entry row
fn create_password_entry_row() -> adwaita::PasswordEntryRow {
    adwaita::PasswordEntryRow::builder()
        .title("Password")
        .build()
}

// Entry row with apply button
fn create_entry_with_apply() -> adwaita::EntryRow {
    let entry = adwaita::EntryRow::builder()
        .title("Server URL")
        .show_apply_button(true)
        .build();
    
    entry.connect_apply(|entry| {
        let text = entry.text();
        println!("Apply: {}", text);
    });
    
    entry
}
```

**Python Example**:

```python
def create_entry_row(self):
    """Create a simple entry row."""
    return Adw.EntryRow(title="Username")

def create_validated_entry_row(self):
    """Create an entry row with validation."""
    entry = Adw.EntryRow(title="Email Address")
    
    # Add suffix icon
    icon = Gtk.Image.new_from_icon_name("emblem-ok-symbolic")
    icon.set_visible(False)
    entry.add_suffix(icon)
    
    # Validate on change
    def on_changed(entry):
        text = entry.get_text()
        is_valid = '@' in text and '.' in text
        icon.set_visible(is_valid)
        
        if is_valid:
            entry.remove_css_class('error')
        elif text:
            entry.add_css_class('error')
        else:
            entry.remove_css_class('error')
    
    entry.connect('changed', on_changed)
    
    return entry

def create_password_entry_row(self):
    """Create a password entry row."""
    return Adw.PasswordEntryRow(title="Password")

def create_entry_with_apply(self):
    """Create an entry row with apply button."""
    entry = Adw.EntryRow(
        title="Server URL",
        show_apply_button=True
    )
    
    entry.connect('apply', self.on_url_apply)
    
    return entry

def on_url_apply(self, entry):
    """Handle apply button click."""
    text = entry.get_text()
    print(f"Apply: {text}")
```

### AdwSpinRow

Spin rows provide numeric input with increment/decrement buttons.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_spin_row() -> adwaita::SpinRow {
    let adjustment = gtk::Adjustment::new(
        5.0,    // initial value
        0.0,    // minimum
        100.0,  // maximum
        1.0,    // step increment
        10.0,   // page increment
        0.0,    // page size
    );
    
    adwaita::SpinRow::builder()
        .title("Volume")
        .subtitle("Adjust the volume level")
        .adjustment(&adjustment)
        .digits(0) // No decimal places
        .build()
}

// Spin row with decimal values
fn create_decimal_spin_row() -> adwaita::SpinRow {
    let adjustment = gtk::Adjustment::new(1.0, 0.0, 10.0, 0.1, 1.0, 0.0);
    
    let spin = adwaita::SpinRow::builder()
        .title("Playback Speed")
        .adjustment(&adjustment)
        .digits(1) // One decimal place
        .build();
    
    // Connect to value change
    spin.connect_output(|spin| {
        let value = spin.value();
        println!("Speed: {:.1}x", value);
        glib::Propagation::Proceed
    });
    
    spin
}

// Spin row with wrap
fn create_spin_row_with_wrap() -> adwaita::SpinRow {
    let adjustment = gtk::Adjustment::new(1.0, 1.0, 12.0, 1.0, 1.0, 0.0);
    
    adwaita::SpinRow::builder()
        .title("Month")
        .adjustment(&adjustment)
        .wrap(true) // Wrap from 12 to 1
        .digits(0)
        .build()
}
```

**Python Example**:

```python
def create_spin_row(self):
    """Create a spin row for integer values."""
    adjustment = Gtk.Adjustment(
        value=5,
        lower=0,
        upper=100,
        step_increment=1,
        page_increment=10,
        page_size=0
    )
    
    return Adw.SpinRow(
        title="Volume",
        subtitle="Adjust the volume level",
        adjustment=adjustment,
        digits=0
    )

def create_decimal_spin_row(self):
    """Create a spin row with decimal values."""
    adjustment = Gtk.Adjustment(
        value=1.0,
        lower=0.0,
        upper=10.0,
        step_increment=0.1,
        page_increment=1.0,
        page_size=0
    )
    
    spin = Adw.SpinRow(
        title="Playback Speed",
        adjustment=adjustment,
        digits=1
    )
    
    # Connect to value change
    spin.connect('output', self.on_speed_changed)
    
    return spin

def on_speed_changed(self, spin):
    """Handle speed value change."""
    value = spin.get_value()
    print(f"Speed: {value:.1f}x")
    return False

def create_spin_row_with_wrap(self):
    """Create a spin row with wrapping."""
    adjustment = Gtk.Adjustment(
        value=1,
        lower=1,
        upper=12,
        step_increment=1,
        page_increment=1,
        page_size=0
    )
    
    return Adw.SpinRow(
        title="Month",
        adjustment=adjustment,
        wrap=True,
        digits=0
    )
```

### AdwSwitchRow

Switch rows provide a toggle switch within a list row.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_switch_row() -> adwaita::SwitchRow {
    let switch_row = adwaita::SwitchRow::builder()
        .title("Wi-Fi")
        .subtitle("Connect to wireless networks")
        .build();
    
    // Connect to activation
    switch_row.connect_active_notify(|row| {
        let active = row.is_active();
        println!("Wi-Fi: {}", if active { "On" } else { "Off" });
    });
    
    switch_row
}

// Switch row with icon
fn create_switch_row_with_icon() -> adwaita::SwitchRow {
    let row = adwaita::SwitchRow::builder()
        .title("Bluetooth")
        .subtitle("Connect to devices")
        .build();
    
    let icon = gtk::Image::from_icon_name("bluetooth-symbolic");
    row.add_prefix(&icon);
    
    row
}

// Bind switch row to GSettings
fn create_bound_switch_row(settings: &gtk::gio::Settings) -> adwaita::SwitchRow {
    let row = adwaita::SwitchRow::builder()
        .title("Dark Mode")
        .subtitle("Use dark appearance")
        .build();
    
    // Bind to settings
    settings
        .bind("dark-mode", &row, "active")
        .build();
    
    row
}
```

**Python Example**:

```python
def create_switch_row(self):
    """Create a simple switch row."""
    switch_row = Adw.SwitchRow(
        title="Wi-Fi",
        subtitle="Connect to wireless networks"
    )
    
    # Connect to activation
    switch_row.connect('notify::active', self.on_wifi_toggled)
    
    return switch_row

def on_wifi_toggled(self, row, _pspec):
    """Handle Wi-Fi toggle."""
    active = row.get_active()
    print(f"Wi-Fi: {'On' if active else 'Off'}")

def create_switch_row_with_icon(self):
    """Create a switch row with icon."""
    row = Adw.SwitchRow(
        title="Bluetooth",
        subtitle="Connect to devices"
    )
    
    icon = Gtk.Image.new_from_icon_name("bluetooth-symbolic")
    row.add_prefix(icon)
    
    return row

def create_bound_switch_row(self, settings):
    """Create a switch row bound to GSettings."""
    row = Adw.SwitchRow(
        title="Dark Mode",
        subtitle="Use dark appearance"
    )
    
    # Bind to settings
    settings.bind('dark-mode', row, 'active',
                  Gio.SettingsBindFlags.DEFAULT)
    
    return row
```



## Chapter 7: Complex Widgets

### AdwNavigationSplitView

The navigation split view is the foundation for sidebar-based navigation.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_navigation_split_view() -> adwaita::NavigationSplitView {
    let split_view = adwaita::NavigationSplitView::new();
    
    // Create sidebar
    let sidebar = create_sidebar();
    let sidebar_page = adwaita::NavigationPage::builder()
        .title("Sidebar")
        .child(&sidebar)
        .build();
    
    // Create content
    let content = create_content();
    let content_page = adwaita::NavigationPage::builder()
        .title("Content")
        .child(&content)
        .build();
    
    split_view.set_sidebar(Some(&sidebar_page));
    split_view.set_content(Some(&content_page));
    
    // Configure behavior
    split_view.set_show_content(true);
    split_view.set_min_sidebar_width(200.0);
    split_view.set_max_sidebar_width(300.0);
    split_view.set_sidebar_width_fraction(0.25);
    
    split_view
}

fn create_sidebar() -> gtk::Widget {
    let list_box = gtk::ListBox::new();
    list_box.add_css_class("navigation-sidebar");
    
    let items = vec!["Inbox", "Sent", "Drafts", "Trash"];
    for item in items {
        let row = adwaita::ActionRow::builder()
            .title(item)
            .activatable(true)
            .build();
        list_box.append(&row);
    }
    
    let scrolled = gtk::ScrolledWindow::new();
    scrolled.set_child(Some(&list_box));
    scrolled.set_vexpand(true);
    
    scrolled.upcast()
}

fn create_content() -> gtk::Widget {
    adwaita::StatusPage::builder()
        .title("No Selection")
        .description("Select an item from the sidebar")
        .icon_name("mail-inbox-symbolic")
        .build()
        .upcast()
}

// Handle navigation
fn setup_navigation(split_view: &adwaita::NavigationSplitView, list: &gtk::ListBox) {
    list.connect_row_activated(glib::clone!(
        #[weak]
        split_view,
        move |_, row| {
            // On mobile, hide sidebar when item selected
            if split_view.is_collapsed() {
                split_view.set_show_content(true);
            }
        }
    ));
}
```

**Python Example**:

```python
def create_navigation_split_view(self):
    """Create a navigation split view with sidebar."""
    split_view = Adw.NavigationSplitView()
    
    # Create sidebar
    sidebar = self.create_sidebar()
    sidebar_page = Adw.NavigationPage(
        title="Sidebar",
        child=sidebar
    )
    
    # Create content
    content = self.create_content()
    content_page = Adw.NavigationPage(
        title="Content",
        child=content
    )
    
    split_view.set_sidebar(sidebar_page)
    split_view.set_content(content_page)
    
    # Configure behavior
    split_view.set_show_content(True)
    split_view.set_min_sidebar_width(200)
    split_view.set_max_sidebar_width(300)
    split_view.set_sidebar_width_fraction(0.25)
    
    return split_view

def create_sidebar(self):
    """Create sidebar content."""
    list_box = Gtk.ListBox()
    list_box.add_css_class('navigation-sidebar')
    
    items = ["Inbox", "Sent", "Drafts", "Trash"]
    for item in items:
        row = Adw.ActionRow(
            title=item,
            activatable=True
        )
        list_box.append(row)
    
    scrolled = Gtk.ScrolledWindow()
    scrolled.set_child(list_box)
    scrolled.set_vexpand(True)
    
    # Connect navigation
    list_box.connect('row-activated', self.on_sidebar_item_activated)
    
    return scrolled

def create_content(self):
    """Create initial content view."""
    return Adw.StatusPage(
        title="No Selection",
        description="Select an item from the sidebar",
        icon_name="mail-inbox-symbolic"
    )

def on_sidebar_item_activated(self, list_box, row):
    """Handle sidebar item activation."""
    # On mobile, hide sidebar when item selected
    if self.split_view.get_collapsed():
        self.split_view.set_show_content(True)
```

### AdwNavigationView

Navigation view manages a stack of pages with back navigation.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_navigation_view() -> adwaita::NavigationView {
    let nav_view = adwaita::NavigationView::new();
    
    // Create root page
    let root_page = create_root_page();
    nav_view.add(&root_page);
    
    nav_view
}

fn create_root_page() -> adwaita::NavigationPage {
    let list = gtk::ListBox::new();
    list.add_css_class("boxed-list");
    
    for i in 1..=10 {
        let row = adwaita::ActionRow::builder()
            .title(&format!("Item {}", i))
            .activatable(true)
            .build();
        
        let chevron = gtk::Image::from_icon_name("go-next-symbolic");
        row.add_suffix(&chevron);
        
        list.append(&row);
    }
    
    let clamp = adwaita::Clamp::new();
    clamp.set_child(Some(&list));
    clamp.set_maximum_size(600);
    
    let scrolled = gtk::ScrolledWindow::new();
    scrolled.set_child(Some(&clamp));
    
    let toolbar_view = adwaita::ToolbarView::new();
    toolbar_view.set_content(Some(&scrolled));
    
    let header = adwaita::HeaderBar::new();
    toolbar_view.add_top_bar(&header);
    
    adwaita::NavigationPage::builder()
        .title("Items")
        .child(&toolbar_view)
        .build()
}

// Navigate to detail page
fn navigate_to_detail(nav_view: &adwaita::NavigationView, item_id: i32) {
    let detail_page = create_detail_page(item_id);
    nav_view.push(&detail_page);
}

fn create_detail_page(item_id: i32) -> adwaita::NavigationPage {
    let content = adwaita::StatusPage::builder()
        .title(&format!("Item {}", item_id))
        .description("Details about this item")
        .build();
    
    let toolbar_view = adwaita::ToolbarView::new();
    toolbar_view.set_content(Some(&content));
```rust
    let header = adwaita::HeaderBar::new();
    toolbar_view.add_top_bar(&header);
    
    adwaita::NavigationPage::builder()
        .title(&format!("Item {}", item_id))
        .child(&toolbar_view)
        .build()
}

// Navigate back programmatically
fn navigate_back(nav_view: &adwaita::NavigationView) {
    if nav_view.navigation_stack().n_items() > 1 {
        nav_view.pop();
    }
}

// Pop to root
fn pop_to_root(nav_view: &adwaita::NavigationView) {
    nav_view.pop_to_tag("root");
}

// Replace current page
fn replace_page(nav_view: &adwaita::NavigationView) {
    let new_page = create_detail_page(99);
    nav_view.replace(&[new_page]);
}
```

**Python Example**:

```python
def create_navigation_view(self):
    """Create a navigation view with root page."""
    self.nav_view = Adw.NavigationView()
    
    # Create root page
    root_page = self.create_root_page()
    self.nav_view.add(root_page)
    
    return self.nav_view

def create_root_page(self):
    """Create the root page with list."""
    list_box = Gtk.ListBox()
    list_box.add_css_class('boxed-list')
    
    for i in range(1, 11):
        row = Adw.ActionRow(
            title=f"Item {i}",
            activatable=True
        )
        
        chevron = Gtk.Image.new_from_icon_name("go-next-symbolic")
        row.add_suffix(chevron)
        
        # Store item ID for later
        row.item_id = i
        
        list_box.append(row)
    
    # Connect navigation
    list_box.connect('row-activated', self.on_item_activated)
    
    clamp = Adw.Clamp()
    clamp.set_child(list_box)
    clamp.set_maximum_size(600)
    
    scrolled = Gtk.ScrolledWindow()
    scrolled.set_child(clamp)
    
    toolbar_view = Adw.ToolbarView()
    toolbar_view.set_content(scrolled)
    
    header = Adw.HeaderBar()
    toolbar_view.add_top_bar(header)
    
    return Adw.NavigationPage(
        title="Items",
        child=toolbar_view
    )

def on_item_activated(self, list_box, row):
    """Navigate to detail page."""
    item_id = row.item_id
    self.navigate_to_detail(item_id)

def navigate_to_detail(self, item_id):
    """Navigate to detail page."""
    detail_page = self.create_detail_page(item_id)
    self.nav_view.push(detail_page)

def create_detail_page(self, item_id):
    """Create detail page for an item."""
    content = Adw.StatusPage(
        title=f"Item {item_id}",
        description="Details about this item"
    )
    
    toolbar_view = Adw.ToolbarView()
    toolbar_view.set_content(content)
    
    header = Adw.HeaderBar()
    toolbar_view.add_top_bar(header)
    
    return Adw.NavigationPage(
        title=f"Item {item_id}",
        child=toolbar_view
    )

def navigate_back(self):
    """Navigate back programmatically."""
    if self.nav_view.get_navigation_stack().get_n_items() > 1:
        self.nav_view.pop()

def pop_to_root(self):
    """Pop to root page."""
    self.nav_view.pop_to_tag("root")

def replace_page(self):
    """Replace current page."""
    new_page = self.create_detail_page(99)
    self.nav_view.replace([new_page])
```

### AdwViewStack and AdwViewSwitcher

View stacks manage multiple views with switchers for navigation.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_view_stack_with_switcher() -> gtk::Box {
    let view_stack = adwaita::ViewStack::new();
    
    // Add pages
    let page1 = create_page_content("Home", "house-symbolic");
    view_stack.add_titled_with_icon(
        &page1,
        Some("home"),
        "Home",
        "house-symbolic"
    );
    
    let page2 = create_page_content("Library", "library-symbolic");
    view_stack.add_titled_with_icon(
        &page2,
        Some("library"),
        "Library",
        "library-symbolic"
    );
    
    let page3 = create_page_content("Settings", "settings-symbolic");
    view_stack.add_titled_with_icon(
        &page3,
        Some("settings"),
        "Settings",
        "settings-symbolic"
    );
    
    // Create switcher bar (bottom navigation on mobile)
    let switcher_bar = adwaita::ViewSwitcherBar::new();
    switcher_bar.set_stack(Some(&view_stack));
    
    // Create switcher title (top navigation on desktop)
    let switcher_title = adwaita::ViewSwitcherTitle::new();
    switcher_title.set_stack(Some(&view_stack));
    
    // Create header
    let header = adwaita::HeaderBar::new();
    header.set_title_widget(Some(&switcher_title));
    
    // Create toolbar view
    let toolbar_view = adwaita::ToolbarView::new();
    toolbar_view.add_top_bar(&header);
    toolbar_view.set_content(Some(&view_stack));
    toolbar_view.add_bottom_bar(&switcher_bar);
    
    // Bind view switcher visibility
    switcher_title
        .bind_property("title-visible", &switcher_bar, "reveal")
        .flags(glib::BindingFlags::SYNC_CREATE)
        .build();
    
    let container = gtk::Box::new(gtk::Orientation::Vertical, 0);
    container.append(&toolbar_view);
    container
}

fn create_page_content(title: &str, icon: &str) -> gtk::Widget {
    adwaita::StatusPage::builder()
        .title(title)
        .icon_name(icon)
        .description(&format!("This is the {} page", title.to_lowercase()))
        .build()
        .upcast()
}

// Change view programmatically
fn change_view(stack: &adwaita::ViewStack, page_name: &str) {
    stack.set_visible_child_name(page_name);
}

// Get current view
fn get_current_view(stack: &adwaita::ViewStack) -> Option<String> {
    stack.visible_child_name().map(|s| s.to_string())
}
```

**Python Example**:

```python
def create_view_stack_with_switcher(self):
    """Create view stack with switcher navigation."""
    self.view_stack = Adw.ViewStack()
    
    # Add pages
    page1 = self.create_page_content("Home", "house-symbolic")
    self.view_stack.add_titled_with_icon(
        page1, "home", "Home", "house-symbolic"
    )
    
    page2 = self.create_page_content("Library", "library-symbolic")
    self.view_stack.add_titled_with_icon(
        page2, "library", "Library", "library-symbolic"
    )
    
    page3 = self.create_page_content("Settings", "settings-symbolic")
    self.view_stack.add_titled_with_icon(
        page3, "settings", "Settings", "settings-symbolic"
    )
    
    # Create switcher bar (bottom navigation on mobile)
    switcher_bar = Adw.ViewSwitcherBar()
    switcher_bar.set_stack(self.view_stack)
    
    # Create switcher title (top navigation on desktop)
    switcher_title = Adw.ViewSwitcherTitle()
    switcher_title.set_stack(self.view_stack)
    
    # Create header
    header = Adw.HeaderBar()
    header.set_title_widget(switcher_title)
    
    # Create toolbar view
    toolbar_view = Adw.ToolbarView()
    toolbar_view.add_top_bar(header)
    toolbar_view.set_content(self.view_stack)
    toolbar_view.add_bottom_bar(switcher_bar)
    
    # Bind view switcher visibility
    switcher_title.bind_property(
        "title-visible", switcher_bar, "reveal",
        GObject.BindingFlags.SYNC_CREATE
    )
    
    return toolbar_view

def create_page_content(self, title, icon):
    """Create content for a page."""
    return Adw.StatusPage(
        title=title,
        icon_name=icon,
        description=f"This is the {title.lower()} page"
    )

def change_view(self, page_name):
    """Change view programmatically."""
    self.view_stack.set_visible_child_name(page_name)

def get_current_view(self):
    """Get current view name."""
    return self.view_stack.get_visible_child_name()
```

### AdwTabBar and AdwTabView

Tab bars provide browser-style tabbed navigation.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_tab_view() -> gtk::Box {
    let tab_view = adwaita::TabView::new();
    
    // Create tab bar
    let tab_bar = adwaita::TabBar::new();
    tab_bar.set_view(Some(&tab_view));
    tab_bar.set_autohide(false); // Always show tab bar
    
    // Add initial tab
    add_new_tab(&tab_view, "Welcome", None);
    
    // Create header
    let header = adwaita::HeaderBar::new();
    
    // Add new tab button
    let new_tab_button = gtk::Button::from_icon_name("tab-new-symbolic");
    new_tab_button.set_tooltip_text(Some("New Tab"));
    new_tab_button.connect_clicked(glib::clone!(
        #[weak]
        tab_view,
        move |_| {
            let count = tab_view.n_pages();
            add_new_tab(&tab_view, &format!("Tab {}", count + 1), None);
        }
    ));
    header.pack_end(&new_tab_button);
    
    // Create toolbar view
    let toolbar_view = adwaita::ToolbarView::new();
    toolbar_view.add_top_bar(&header);
    toolbar_view.add_top_bar(&tab_bar);
    toolbar_view.set_content(Some(&tab_view));
    
    // Handle tab close
    tab_view.connect_close_page(|_, page| {
        // Return true to allow closing, false to prevent
        true.into()
    });
    
    let container = gtk::Box::new(gtk::Orientation::Vertical, 0);
    container.append(&toolbar_view);
    container
}

fn add_new_tab(tab_view: &adwaita::TabView, title: &str, icon: Option<&str>) {
    let content = adwaita::StatusPage::builder()
        .title(title)
        .icon_name(icon.unwrap_or("document-symbolic"))
        .build();
    
    let page = tab_view.append(&content);
    
    // Set tab properties
    tab_view.set_page_title(&page, title);
    if let Some(icon_name) = icon {
        tab_view.set_page_icon(&page, Some(&gtk::gio::ThemedIcon::new(icon_name)));
    }
    
    // Select the new tab
    tab_view.set_selected_page(&page);
}

// Tab operations
fn close_current_tab(tab_view: &adwaita::TabView) {
    if let Some(page) = tab_view.selected_page() {
        tab_view.close_page(&page);
    }
}

fn close_all_tabs(tab_view: &adwaita::TabView) {
    tab_view.close_pages_after(None);
}

fn pin_current_tab(tab_view: &adwaita::TabView) {
    if let Some(page) = tab_view.selected_page() {
        let is_pinned = tab_view.page_is_pinned(&page);
        tab_view.set_page_pinned(&page, !is_pinned);
    }
}

// Tab reordering
fn move_tab_left(tab_view: &adwaita::TabView) {
    if let Some(page) = tab_view.selected_page() {
        let position = tab_view.page_position(&page);
        if position > 0 {
            tab_view.reorder_page(&page, position - 1);
        }
    }
}
```

**Python Example**:

```python
def create_tab_view(self):
    """Create a tab view with tab bar."""
    self.tab_view = Adw.TabView()
    
    # Create tab bar
    tab_bar = Adw.TabBar()
    tab_bar.set_view(self.tab_view)
    tab_bar.set_autohide(False)
    
    # Add initial tab
    self.add_new_tab("Welcome")
    
    # Create header
    header = Adw.HeaderBar()
    
    # Add new tab button
    new_tab_button = Gtk.Button.new_from_icon_name("tab-new-symbolic")
    new_tab_button.set_tooltip_text("New Tab")
    new_tab_button.connect('clicked', self.on_new_tab_clicked)
    header.pack_end(new_tab_button)
    
    # Create toolbar view
    toolbar_view = Adw.ToolbarView()
    toolbar_view.add_top_bar(header)
    toolbar_view.add_top_bar(tab_bar)
    toolbar_view.set_content(self.tab_view)
    
    # Handle tab close
    self.tab_view.connect('close-page', self.on_tab_close)
    
    return toolbar_view

def on_new_tab_clicked(self, button):
    """Create a new tab."""
    count = self.tab_view.get_n_pages()
    self.add_new_tab(f"Tab {count + 1}")

def add_new_tab(self, title, icon=None):
    """Add a new tab to the tab view."""
    content = Adw.StatusPage(
        title=title,
        icon_name=icon or "document-symbolic"
    )
    
    page = self.tab_view.append(content)
    
    # Set tab properties
    self.tab_view.set_page_title(page, title)
    if icon:
        themed_icon = Gio.ThemedIcon.new(icon)
        self.tab_view.set_page_icon(page, themed_icon)
    
    # Select the new tab
    self.tab_view.set_selected_page(page)

def on_tab_close(self, tab_view, page):
    """Handle tab close request."""
    # Return True to allow closing, False to prevent
    return True

def close_current_tab(self):
    """Close the currently selected tab."""
    page = self.tab_view.get_selected_page()
    if page:
        self.tab_view.close_page(page)

def close_all_tabs(self):
    """Close all tabs."""
    self.tab_view.close_pages_after(None)

def pin_current_tab(self):
    """Toggle pin state of current tab."""
    page = self.tab_view.get_selected_page()
    if page:
        is_pinned = self.tab_view.get_page_is_pinned(page)
        self.tab_view.set_page_pinned(page, not is_pinned)

def move_tab_left(self):
    """Move current tab to the left."""
    page = self.tab_view.get_selected_page()
    if page:
        position = self.tab_view.get_page_position(page)
        if position > 0:
            self.tab_view.reorder_page(page, position - 1)
```

### AdwCarousel

Carousels provide swipeable page navigation.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_carousel() -> adwaita::Carousel {
    let carousel = adwaita::Carousel::new();
    carousel.set_interactive(true);
    carousel.set_allow_long_swipes(true);
    
    // Add pages
    for i in 1..=5 {
        let page = create_carousel_page(i);
        carousel.append(&page);
    }
    
    carousel
}

fn create_carousel_page(number: i32) -> gtk::Widget {
    let page = adwaita::StatusPage::builder()
        .title(&format!("Page {}", number))
        .description(&format!("This is page {} of 5", number))
        .icon_name("document-symbolic")
        .build();
    
    page.upcast()
}

// With indicator dots
fn create_carousel_with_indicator() -> gtk::Box {
    let carousel = create_carousel();
    
    let indicator = adwaita::CarouselIndicatorDots::new();
    indicator.set_carousel(Some(&carousel));
    
    let container = gtk::Box::new(gtk::Orientation::Vertical, 12);
    container.append(&carousel);
    container.append(&indicator);
    container.set_vexpand(true);
    
    container
}

// With line indicator
fn create_carousel_with_lines() -> gtk::Box {
    let carousel = create_carousel();
    
    let indicator = adwaita::CarouselIndicatorLines::new();
    indicator.set_carousel(Some(&carousel));
    
    let container = gtk::Box::new(gtk::Orientation::Vertical, 12);
    container.append(&carousel);
    container.append(&indicator);
    container.set_vexpand(true);
    
    container
}

// Navigate carousel programmatically
fn scroll_to_page(carousel: &adwaita::Carousel, position: f64) {
    carousel.scroll_to(&carousel.nth_page(position as u32), true);
}

// Get current page
fn get_current_page(carousel: &adwaita::Carousel) -> f64 {
    carousel.position()
}

// Listen to page changes
fn setup_carousel_listener(carousel: &adwaita::Carousel) {
    carousel.connect_position_notify(|carousel| {
        let position = carousel.position();
        println!("Current page: {}", position);
    });
}
```

**Python Example**:

```python
def create_carousel(self):
    """Create a carousel with multiple pages."""
    carousel = Adw.Carousel()
    carousel.set_interactive(True)
    carousel.set_allow_long_swipes(True)
    
    # Add pages
    for i in range(1, 6):
        page = self.create_carousel_page(i)
        carousel.append(page)
    
    return carousel

def create_carousel_page(self, number):
    """Create a carousel page."""
    return Adw.StatusPage(
        title=f"Page {number}",
        description=f"This is page {number} of 5",
        icon_name="document-symbolic"
    )

def create_carousel_with_indicator(self):
    """Create carousel with dot indicators."""
    carousel = self.create_carousel()
    
    indicator = Adw.CarouselIndicatorDots()
    indicator.set_carousel(carousel)
    
    container = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=12)
    container.append(carousel)
    container.append(indicator)
    container.set_vexpand(True)
    
    return container

def create_carousel_with_lines(self):
    """Create carousel with line indicators."""
    carousel = self.create_carousel()
    
    indicator = Adw.CarouselIndicatorLines()
    indicator.set_carousel(carousel)
    
    container = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=12)
    container.append(carousel)
    container.append(indicator)
    container.set_vexpand(True)
    
    return container

def scroll_to_page(self, carousel, position):
    """Navigate carousel to specific page."""
    page = carousel.get_nth_page(int(position))
    carousel.scroll_to(page, True)

def get_current_page(self, carousel):
    """Get current page position."""
    return carousel.get_position()

def setup_carousel_listener(self, carousel):
    """Listen to carousel page changes."""
    carousel.connect('notify::position', self.on_carousel_position_changed)

def on_carousel_position_changed(self, carousel, _pspec):
    """Handle carousel position change."""
    position = carousel.get_position()
    print(f"Current page: {position}")
```

### AdwPreferencesWindow

Preferences windows provide a standardized settings interface.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_preferences_window(parent: &impl IsA<gtk::Window>) -> adwaita::PreferencesWindow {
    let prefs = adwaita::PreferencesWindow::new();
    prefs.set_transient_for(Some(parent));
    prefs.set_modal(true);
    prefs.set_search_enabled(true);
    
    // Add pages
    let general_page = create_general_page();
    prefs.add(&general_page);
    
    let appearance_page = create_appearance_page();
    prefs.add(&appearance_page);
    
    let advanced_page = create_advanced_page();
    prefs.add(&advanced_page);
    
    prefs
}

fn create_general_page() -> adwaita::PreferencesPage {
    let page = adwaita::PreferencesPage::builder()
        .title("General")
        .icon_name("preferences-system-symbolic")
        .build();
    
    // Create group
    let group = adwaita::PreferencesGroup::builder()
        .title("Application")
        .description("Configure application behavior")
        .build();
    
    // Add rows
    let auto_save = adwaita::SwitchRow::builder()
        .title("Auto Save")
        .subtitle("Automatically save changes")
        .build();
    group.add(&auto_save);
    
    let interval = adwaita::SpinRow::builder()
        .title("Save Interval")
        .subtitle("Minutes between auto-saves")
        .adjustment(&gtk::Adjustment::new(5.0, 1.0, 60.0, 1.0, 5.0, 0.0))
        .build();
    group.add(&interval);
    
    page.add(&group);
    
    // Another group
    let files_group = adwaita::PreferencesGroup::builder()
        .title("Files")
        .build();
    
    let default_folder = adwaita::ActionRow::builder()
        .title("Default Folder")
        .subtitle("/home/user/Documents")
        .activatable(true)
        .build();
    let chevron = gtk::Image::from_icon_name("go-next-symbolic");
    default_folder.add_suffix(&chevron);
    files_group.add(&default_folder);
    
    page.add(&files_group);
    
    page
}

fn create_appearance_page() -> adwaita::PreferencesPage {
    let page = adwaita::PreferencesPage::builder()
        .title("Appearance")
        .icon_name("preferences-desktop-theme-symbolic")
        .build();
    
    let group = adwaita::PreferencesGroup::builder()
        .title("Theme")
        .build();
    
    // Color scheme selection
    let color_scheme = adwaita::ComboRow::builder()
        .title("Color Scheme")
        .subtitle("Choose light or dark theme")
        .model(&gtk::StringList::new(&["Light", "Dark", "Follow System"]))
        .selected(2)
        .build();
    group.add(&color_scheme);
    
    // Font size
    let font_size = adwaita::SpinRow::builder()
        .title("Font Size")
        .adjustment(&gtk::Adjustment::new(11.0, 8.0, 24.0, 1.0, 2.0, 0.0))
        .digits(0)
        .build();
    group.add(&font_size);
    
    page.add(&group);
    page
}

fn create_advanced_page() -> adwaita::PreferencesPage {
    let page = adwaita::PreferencesPage::builder()
        .title("Advanced")
        .icon_name("preferences-other-symbolic")
        .build();
    
    let group = adwaita::PreferencesGroup::builder()
        .title("Developer Options")
        .build();
    
    let debug_mode = adwaita::SwitchRow::builder()
        .title("Debug Mode")
        .subtitle("Enable verbose logging")
        .build();
    group.add(&debug_mode);
    
    let expander = adwaita::ExpanderRow::builder()
        .title("Cache Settings")
        .build();
    
    let clear_cache = adwaita::ActionRow::builder()
        .title("Clear Cache")
        .subtitle("Remove all cached data")
        .activatable(true)
        .build();
    let delete_icon = gtk::Image::from_icon_name("user-trash-symbolic");
    delete_icon.add_css_class("error");
    clear_cache.add_suffix(&delete_icon);
    expander.add_row(&clear_cache);
    
    group.add(&expander);
    page.add(&group);
    
    page
}
```

**Python Example**:

```python
def create_preferences_window(self, parent):
    """Create a preferences window."""
    prefs = Adw.PreferencesWindow()
    prefs.set_transient_for(parent)
    prefs.set_modal(True)
    prefs.set_search_enabled(True)
    
    # Add pages
    prefs.add(self.create_general_page())
    prefs.add(self.create_appearance_page())
    prefs.add(self.create_advanced_page())
    
    return prefs

def create_general_page(self):
    """Create general preferences page."""
    page = Adw.PreferencesPage(
        title="General",
        icon_name="preferences-system-symbolic"
    )
    
    # Create group
    group = Adw.PreferencesGroup(
        title="Application",
        description="Configure application behavior"
    )
    
    # Add rows
    auto_save = Adw.SwitchRow(
        title="Auto Save",
        subtitle="Automatically save changes"
    )
    group.add(auto_save)
    
    interval = Adw.SpinRow(
        title="Save Interval",
        subtitle="Minutes between auto-saves",
        adjustment=Gtk.Adjustment(
            value=5, lower=1, upper=60,
            step_increment=1, page_increment=5
        )
    )
    group.add(interval)
    
    page.add(group)
    
    # Another group
    files_group = Adw.PreferencesGroup(title="Files")
    
    default_folder = Adw.ActionRow(
        title="Default Folder",
        subtitle="/home/user/Documents",
        activatable=True
    )
    chevron = Gtk.Image.new_from_icon_name("go-next-symbolic")
    default_folder.add_suffix(chevron)
    files_group.add(default_folder)
    
    page.add(files_group)
    
    return page

def create_appearance_page(self):
    """Create appearance preferences page."""
    page = Adw.PreferencesPage(
        title="Appearance",
        icon_name="preferences-desktop-theme-symbolic"
    )
    
    group = Adw.PreferencesGroup(title="Theme")
    
    # Color scheme selection
    color_scheme = Adw.ComboRow(
        title="Color Scheme",
        subtitle="Choose light or dark theme",
        model=Gtk.StringList.new(["Light", "Dark", "Follow System"]),
        selected=2
    )
    group.add(color_scheme)
    
    # Font size
    font_size = Adw.SpinRow(
        title="Font Size",
        adjustment=Gtk.Adjustment(
            value=11, lower=8, upper=24,
            step_increment=1, page_increment=2
        ),
        digits=0
    )
    group.add(font_size)
    
    page.add(group)
    return page

def create_advanced_page(self):
    """Create advanced preferences page."""
    page = Adw.PreferencesPage(
        title="Advanced",
        icon_name="preferences-other-symbolic"
    )
    
    group = Adw.PreferencesGroup(title="Developer Options")
    
    debug_mode = Adw.SwitchRow(
        title="Debug Mode",
        subtitle="Enable verbose logging"
    )
    group.add(debug_mode)
    
    expander = Adw.ExpanderRow(title="Cache Settings")
    
    clear_cache = Adw.ActionRow(
        title="Clear Cache",
        subtitle="Remove all cached data",
        activatable=True
    )
    delete_icon = Gtk.Image.new_from_icon_name("user-trash-symbolic")
    delete_icon.add_css_class("error")
    clear_cache.add_suffix(delete_icon)
    expander.add_row(clear_cache)
    
    group.add(expander)
    page.add(group)
    
    return page
```



## Chapter 8: Layouts and Containers

### AdwClamp

Clamps limit content width for better readability.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_clamped_content() -> adwaita::Clamp {
    let content = gtk::Label::new(Some(
        "This text will be clamped to a maximum width for better readability. \
        On wide screens, it won't stretch too far."
    ));
    content.set_wrap(true);
    content.set_xalign(0.0);
    
    let clamp = adwaita::Clamp::new();
    clamp.set_child(Some(&content));
    clamp.set_maximum_size(600); // Maximum width in pixels
    clamp.set_tightening_threshold(400); // Start tightening below this width
    
    clamp
}

// Clamp in scrolled window
fn create_scrolled_clamped_content() -> gtk::ScrolledWindow {
    let list = gtk::ListBox::new();
    list.add_css_class("boxed-list");
    
    for i in 1..=20 {
        let row = adwaita::ActionRow::builder()
            .title(&format!("Item {}", i))
            .build();
        list.append(&row);
    }
    
    let clamp = adwaita::Clamp::new();
    clamp.set_child(Some(&list));
    clamp.set_maximum_size(600);

    let scrolled = gtk::ScrolledWindow::new();
    scrolled.set_child(Some(&clamp));
    scrolled.set_vexpand(true);
    
    scrolled
}
```

**Python Example**:

```python
def create_clamped_content(self):
    """Create content with width clamping."""
    content = Gtk.Label(
        label="This text will be clamped to a maximum width for better readability. "
              "On wide screens, it won't stretch too far.",
        wrap=True,
        xalign=0.0
    )
    
    clamp = Adw.Clamp()
    clamp.set_child(content)
    clamp.set_maximum_size(600)  # Maximum width in pixels
    clamp.set_tightening_threshold(400)  # Start tightening below this width
    
    return clamp

def create_scrolled_clamped_content(self):
    """Create scrolled content with clamping."""
    list_box = Gtk.ListBox()
    list_box.add_css_class('boxed-list')
    
    for i in range(1, 21):
        row = Adw.ActionRow(title=f"Item {i}")
        list_box.append(row)
    
    clamp = Adw.Clamp()
    clamp.set_child(list_box)
    clamp.set_maximum_size(600)
    
    scrolled = Gtk.ScrolledWindow()
    scrolled.set_child(clamp)
    scrolled.set_vexpand(True)
    
    return scrolled
```

### AdwBin

Bins are simple containers that hold a single child.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_bin_example() -> adwaita::Bin {
    let button = gtk::Button::with_label("Click me");
    
    let bin = adwaita::Bin::new();
    bin.set_child(Some(&button));
    
    bin
}

// Bin with dynamic content switching
fn create_dynamic_bin() -> adwaita::Bin {
    let bin = adwaita::Bin::new();
    
    // Initial content
    let label = gtk::Label::new(Some("Initial content"));
    bin.set_child(Some(&label));
    
    bin
}

fn switch_bin_content(bin: &adwaita::Bin, show_spinner: bool) {
    if show_spinner {
        let spinner = gtk::Spinner::new();
        spinner.start();
        bin.set_child(Some(&spinner));
    } else {
        let label = gtk::Label::new(Some("Content loaded"));
        bin.set_child(Some(&label));
    }
}
```

**Python Example**:

```python
def create_bin_example(self):
    """Create a simple bin container."""
    button = Gtk.Button(label="Click me")
    
    bin_container = Adw.Bin()
    bin_container.set_child(button)
    
    return bin_container

def create_dynamic_bin(self):
    """Create bin with dynamic content."""
    self.bin = Adw.Bin()
    
    # Initial content
    label = Gtk.Label(label="Initial content")
    self.bin.set_child(label)
    
    return self.bin

def switch_bin_content(self, show_spinner):
    """Switch content in bin."""
    if show_spinner:
        spinner = Gtk.Spinner()
        spinner.start()
        self.bin.set_child(spinner)
    else:
        label = Gtk.Label(label="Content loaded")
        self.bin.set_child(label)
```

### AdwToolbarView

Toolbar views manage top and bottom bars around content.

**Rust Example**:

```rust
use adwaita::prelude::*;

fn create_toolbar_view() -> adwaita::ToolbarView {
    let toolbar_view = adwaita::ToolbarView::new();
    
    // Add header bar
    let header = adwaita::HeaderBar::new();
    toolbar_view.add_top_bar(&header);
    
    // Add content
    let content = gtk::Label::new(Some("Main content area"));
    content.set_vexpand(true);
    toolbar_view.set_content(Some(&content));
    
    // Add bottom bar
    let bottom_bar = gtk::ActionBar::new();
    let button = gtk::Button::with_label("Action");
    bottom_bar.pack_start(&button);
    toolbar_view.add_bottom_bar(&bottom_bar);
    
    toolbar_view
}

// Toolbar view with multiple top bars
fn create_multi_bar_view() -> adwaita::ToolbarView {
    let toolbar_view = adwaita::ToolbarView::new();
    
    // Header bar
    let header = adwaita::HeaderBar::new();
    toolbar_view.add_top_bar(&header);
    
    // Search bar
    let search_bar = gtk::SearchBar::new();
    let search_entry = gtk::SearchEntry::new();
    search_bar.set_child(Some(&search_entry));
    toolbar_view.add_top_bar(&search_bar);
    
    // Info bar / Banner
    let banner = adwaita::Banner::new("Updates available");
    banner.set_button_label(Some("Install"));
    toolbar_view.add_top_bar(&banner);
    
    // Content
    let content = gtk::Label::new(Some("Content"));
    content.set_vexpand(true);
    toolbar_view.set_content(Some(&content));
    
    toolbar_view
}

// Control reveal state
fn set_toolbar_revealed(toolbar_view: &adwaita::ToolbarView, revealed: bool) {
    toolbar_view.set_reveal_top_bars(revealed);
    toolbar_view.set_reveal_bottom_bars(revealed);
}

// Extend content into toolbar areas
fn create_extended_toolbar_view() -> adwaita::ToolbarView {
    let toolbar_view = adwaita::ToolbarView::new();
    toolbar_view.set_extend_content_to_top_edge(true);
    toolbar_view.set_extend_content_to_bottom_edge(true);
    
    // This allows content like images to extend under transparent toolbars
    
    let header = adwaita::HeaderBar::new();
    toolbar_view.add_top_bar(&header);
    
    let image = gtk::Picture::for_filename("background.jpg");
    toolbar_view.set_content(Some(&image));
    
    toolbar_view
}
```

**Python Example**:

```python
def create_toolbar_view(self):
    """Create a toolbar view with top and bottom bars."""
    toolbar_view = Adw.ToolbarView()
    
    # Add header bar
    header = Adw.HeaderBar()
    toolbar_view.add_top_bar(header)
    
    # Add content
    content = Gtk.Label(label="Main content area")
    content.set_vexpand(True)
    toolbar_view.set_content(content)
    
    # Add bottom bar
    bottom_bar = Gtk.ActionBar()
    button = Gtk.Button(label="Action")
    bottom_bar.pack_start(button)
    toolbar_view.add_bottom_bar(bottom_bar)
    
    return toolbar_view

def create_multi_bar_view(self):
    """Create toolbar view with multiple top bars."""
    toolbar_view = Adw.ToolbarView()
    
    # Header bar
    header = Adw.HeaderBar()
    toolbar_view.add_top_bar(header)
    
    # Search bar
    search_bar = Gtk.SearchBar()
    search_entry = Gtk.SearchEntry()
    search_bar.set_child(search_entry)
    toolbar_view.add_top_bar(search_bar)
    
    # Banner
    banner = Adw.Banner(title="Updates available")
    banner.set_button_label("Install")
    toolbar_view.add_top_bar(banner)
    
    # Content
    content = Gtk.Label(label="Content")
    content.set_vexpand(True)
    toolbar_view.set_content(content)
    
    return toolbar_view

def set_toolbar_revealed(self, toolbar_view, revealed):
    """Control toolbar reveal state."""
    toolbar_view.set_reveal_top_bars(revealed)
    toolbar_view.set_reveal_bottom_bars(revealed)

def create_extended_toolbar_view(self):
    """Create toolbar view with extended content."""
    toolbar_view = Adw.ToolbarView()
    toolbar_view.set_extend_content_to_top_edge(True)
    toolbar_view.set_extend_content_to_bottom_edge(True)
    
    header = Adw.HeaderBar()
    toolbar_view.add_top_bar(header)
    
    image = Gtk.Picture.new_for_filename("background.jpg")
    toolbar_view.set_content(image)
    
    return toolbar_view
```

### AdwOverlaySplitView

Overlay split views provide sidebar navigation that overlays on mobile.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

fn create_overlay_split_view() -> adwaita::OverlaySplitView {
    let split_view = adwaita::OverlaySplitView::new();
    
    // Create sidebar
    let sidebar = create_sidebar_list();
    split_view.set_sidebar(Some(&sidebar));
    
    // Create content
    let content = create_main_content();
    split_view.set_content(Some(&content));
    
    // Configure
    split_view.set_sidebar_position(gtk::PackType::Start);
    split_view.set_min_sidebar_width(200.0);
    split_view.set_max_sidebar_width(300.0);
    split_view.set_sidebar_width_fraction(0.25);
    
    // Pin sidebar on desktop, overlay on mobile
    split_view.set_pin_sidebar(true);
    
    split_view
}

fn create_sidebar_list() -> gtk::Widget {
    let list = gtk::ListBox::new();
    list.add_css_class("navigation-sidebar");
    
    let items = vec![
        ("Inbox", "mail-inbox-symbolic"),
        ("Sent", "mail-sent-symbolic"),
        ("Drafts", "document-edit-symbolic"),
        ("Trash", "user-trash-symbolic"),
    ];
    
    for (title, icon) in items {
        let row = adwaita::ActionRow::builder()
            .title(title)
            .activatable(true)
            .build();
        
        let icon_widget = gtk::Image::from_icon_name(icon);
        row.add_prefix(&icon_widget);
        
        list.append(&row);
    }
    
    let scrolled = gtk::ScrolledWindow::new();
    scrolled.set_child(Some(&list));
    scrolled.upcast()
}

fn create_main_content() -> gtk::Widget {
    adwaita::StatusPage::builder()
        .title("Select an Item")
        .description("Choose something from the sidebar")
        .icon_name("mail-inbox-symbolic")
        .build()
        .upcast()
}

// Toggle sidebar
fn toggle_sidebar(split_view: &adwaita::OverlaySplitView) {
    let shown = split_view.shows_sidebar();
    split_view.set_show_sidebar(!shown);
}

// Handle adaptive behavior
fn setup_split_view_handlers(split_view: &adwaita::OverlaySplitView) {
    split_view.connect_show_sidebar_notify(|view| {
        let shown = view.shows_sidebar();
        println!("Sidebar is now {}", if shown { "shown" } else { "hidden" });
    });
    
    split_view.connect_collapsed_notify(|view| {
        let collapsed = view.is_collapsed();
        println!("View is {}", if collapsed { "collapsed" } else { "expanded" });
    });
}
```

**Python Example**:

```python
def create_overlay_split_view(self):
    """Create an overlay split view."""
    self.split_view = Adw.OverlaySplitView()
    
    # Create sidebar
    sidebar = self.create_sidebar_list()
    self.split_view.set_sidebar(sidebar)
    
    # Create content
    content = self.create_main_content()
    self.split_view.set_content(content)
    
    # Configure
    self.split_view.set_sidebar_position(Gtk.PackType.START)
    self.split_view.set_min_sidebar_width(200)
    self.split_view.set_max_sidebar_width(300)
    self.split_view.set_sidebar_width_fraction(0.25)
    
    # Pin sidebar on desktop
    self.split_view.set_pin_sidebar(True)
    
    return self.split_view

def create_sidebar_list(self):
    """Create sidebar list."""
    list_box = Gtk.ListBox()
    list_box.add_css_class('navigation-sidebar')
    
    items = [
        ("Inbox", "mail-inbox-symbolic"),
        ("Sent", "mail-sent-symbolic"),
        ("Drafts", "document-edit-symbolic"),
        ("Trash", "user-trash-symbolic"),
    ]
    
    for title, icon in items:
        row = Adw.ActionRow(
            title=title,
            activatable=True
        )
        
        icon_widget = Gtk.Image.new_from_icon_name(icon)
        row.add_prefix(icon_widget)
        
        list_box.append(row)
    
    list_box.connect('row-activated', self.on_sidebar_activated)
    
    scrolled = Gtk.ScrolledWindow()
    scrolled.set_child(list_box)
    return scrolled

def create_main_content(self):
    """Create main content area."""
    return Adw.StatusPage(
        title="Select an Item",
        description="Choose something from the sidebar",
        icon_name="mail-inbox-symbolic"
    )

def toggle_sidebar(self):
    """Toggle sidebar visibility."""
    shown = self.split_view.get_show_sidebar()
    self.split_view.set_show_sidebar(not shown)

def on_sidebar_activated(self, list_box, row):
    """Handle sidebar item activation."""
    # On mobile, hide sidebar after selection
    if self.split_view.get_collapsed():
        self.split_view.set_show_sidebar(False)

def setup_split_view_handlers(self):
    """Setup split view property listeners."""
    self.split_view.connect('notify::show-sidebar', self.on_sidebar_visibility_changed)
    self.split_view.connect('notify::collapsed', self.on_collapse_changed)

def on_sidebar_visibility_changed(self, split_view, _pspec):
    """Handle sidebar visibility change."""
    shown = split_view.get_show_sidebar()
    print(f"Sidebar is now {'shown' if shown else 'hidden'}")

def on_collapse_changed(self, split_view, _pspec):
    """Handle collapse state change."""
    collapsed = split_view.get_collapsed()
    print(f"View is {'collapsed' if collapsed else 'expanded'}")
```

### GTK Box and Grid

Standard GTK containers are still used for layout.

**Rust Example**:

```rust
use gtk::prelude::*;

// Vertical box
fn create_vertical_box() -> gtk::Box {
    let vbox = gtk::Box::new(gtk::Orientation::Vertical, 12);
    vbox.set_margin_top(12);
    vbox.set_margin_bottom(12);
    vbox.set_margin_start(12);
    vbox.set_margin_end(12);
    
    let label = gtk::Label::new(Some("Title"));
    label.add_css_class("title-1");
    vbox.append(&label);
    
    let entry = gtk::Entry::new();
    entry.set_placeholder_text(Some("Enter text..."));
    vbox.append(&entry);
    
    let button = gtk::Button::with_label("Submit");
    button.add_css_class("suggested-action");
    vbox.append(&button);
    
    vbox
}

// Horizontal box
fn create_horizontal_box() -> gtk::Box {
    let hbox = gtk::Box::new(gtk::Orientation::Horizontal, 6);
    hbox.set_homogeneous(true); // Equal width children
    
    for i in 1..=3 {
        let button = gtk::Button::with_label(&format!("Button {}", i));
        hbox.append(&button);
    }
    
    hbox
}

// Grid layout
fn create_grid() -> gtk::Grid {
    let grid = gtk::Grid::new();
    grid.set_row_spacing(12);
    grid.set_column_spacing(12);
    grid.set_margin_top(12);
    grid.set_margin_bottom(12);
    grid.set_margin_start(12);
    grid.set_margin_end(12);
    
    // Labels in first column
    let name_label = gtk::Label::new(Some("Name:"));
    name_label.set_xalign(1.0);
    grid.attach(&name_label, 0, 0, 1, 1);
    
    let email_label = gtk::Label::new(Some("Email:"));
    email_label.set_xalign(1.0);
    grid.attach(&email_label, 0, 1, 1, 1);
    
    // Entries in second column
    let name_entry = gtk::Entry::new();
    name_entry.set_hexpand(true);
    grid.attach(&name_entry, 1, 0, 1, 1);
    
    let email_entry = gtk::Entry::new();
    email_entry.set_hexpand(true);
    grid.attach(&email_entry, 1, 1, 1, 1);
    
    // Button spanning both columns
    let button = gtk::Button::with_label("Submit");
    button.add_css_class("suggested-action");
    grid.attach(&button, 0, 2, 2, 1);
    
    grid
}

// Center box (start, center, end sections)
fn create_center_box() -> gtk::CenterBox {
    let center_box = gtk::CenterBox::new();
    
    // Start
    let back_button = gtk::Button::from_icon_name("go-previous-symbolic");
    center_box.set_start_widget(Some(&back_button));
    
    // Center
    let title = gtk::Label::new(Some("Page Title"));
    title.add_css_class("title-3");
    center_box.set_center_widget(Some(&title));
    
    // End
    let menu_button = gtk::MenuButton::new();
    menu_button.set_icon_name("open-menu-symbolic");
    center_box.set_end_widget(Some(&menu_button));
    
    center_box
}
```

**Python Example**:

```python
def create_vertical_box(self):
    """Create a vertical box layout."""
    vbox = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=12)
    vbox.set_margin_top(12)
    vbox.set_margin_bottom(12)
    vbox.set_margin_start(12)
    vbox.set_margin_end(12)
    
    label = Gtk.Label(label="Title")
    label.add_css_class("title-1")
    vbox.append(label)
    
    entry = Gtk.Entry()
    entry.set_placeholder_text("Enter text...")
    vbox.append(entry)
    
    button = Gtk.Button(label="Submit")
    button.add_css_class("suggested-action")
    vbox.append(button)
    
    return vbox

def create_horizontal_box(self):
    """Create a horizontal box layout."""
    hbox = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL, spacing=6)
    hbox.set_homogeneous(True)  # Equal width children
    
    for i in range(1, 4):
        button = Gtk.Button(label=f"Button {i}")
        hbox.append(button)
    
    return hbox

def create_grid(self):
    """Create a grid layout."""
    grid = Gtk.Grid()
    grid.set_row_spacing(12)
    grid.set_column_spacing(12)
    grid.set_margin_top(12)
    grid.set_margin_bottom(12)
    grid.set_margin_start(12)
    grid.set_margin_end(12)
    
    # Labels in first column
    name_label = Gtk.Label(label="Name:")
    name_label.set_xalign(1.0)
    grid.attach(name_label, 0, 0, 1, 1)
    
    email_label = Gtk.Label(label="Email:")
    email_label.set_xalign(1.0)
    grid.attach(email_label, 0, 1, 1, 1)
    
    # Entries in second column
    name_entry = Gtk.Entry()
    name_entry.set_hexpand(True)
    grid.attach(name_entry, 1, 0, 1, 1)
    
    email_entry = Gtk.Entry()
    email_entry.set_hexpand(True)
    grid.attach(email_entry, 1, 1, 1, 1)
    
    # Button spanning both columns
    button = Gtk.Button(label="Submit")
    button.add_css_class("suggested-action")
    grid.attach(button, 0, 2, 2, 1)
    
    return grid

def create_center_box(self):
    """Create a center box layout."""
    center_box = Gtk.CenterBox()
    
    # Start
    back_button = Gtk.Button.new_from_icon_name("go-previous-symbolic")
    center_box.set_start_widget(back_button)
    
    # Center
    title = Gtk.Label(label="Page Title")
    title.add_css_class("title-3")
    center_box.set_center_widget(title)
    
    # End
    menu_button = Gtk.MenuButton()
    menu_button.set_icon_name("open-menu-symbolic")
    center_box.set_end_widget(menu_button)
    
    return center_box
```



## Chapter 9: Application Patterns

### Dialogs and Alerts

Modern GNOME applications use `AdwAlertDialog` instead of traditional dialogs.

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

// Simple alert dialog
fn show_simple_alert(parent: &impl IsA<gtk::Window>) {
    let dialog = adwaita::AlertDialog::builder()
        .heading("Delete File?")
        .body("This action cannot be undone.")
        .build();
    
    dialog.add_response("cancel", "Cancel");
    dialog.add_response("delete", "Delete");
    
    // Make delete button red/destructive
    dialog.set_response_appearance("delete", adwaita::ResponseAppearance::Destructive);
    
    // Set default response (activated by Enter)
    dialog.set_default_response(Some("cancel"));
    
    // Set close response (activated by Escape)
    dialog.set_close_response("cancel");
    
    dialog.choose(
        Some(parent),
        None::<&gtk::gio::Cancellable>,
        |response| {
            if response == "delete" {
                println!("File deleted");
            }
        },
    );
}

// Alert with extra child widget
fn show_alert_with_extra_content(parent: &impl IsA<gtk::Window>) {
    let checkbox = gtk::CheckButton::with_label("Don't ask again");
    
    let dialog = adwaita::AlertDialog::builder()
        .heading("Clear History?")
        .body("All browsing history will be removed.")
        .extra_child(&checkbox)
        .build();
    
    dialog.add_response("cancel", "Cancel");
    dialog.add_response("clear", "Clear");
    dialog.set_response_appearance("clear", adwaita::ResponseAppearance::Destructive);
    dialog.set_default_response(Some("cancel"));
    dialog.set_close_response("cancel");
    
    dialog.choose(
        Some(parent),
        None::<&gtk::gio::Cancellable>,
        glib::clone!(
            #[weak]
            checkbox,
            move |response| {
                if response == "clear" {
                    let dont_ask = checkbox.is_active();
                    println!("Clear history (don't ask: {})", dont_ask);
                }
            }
        ),
    );
}

// Async dialog
async fn show_async_alert(parent: gtk::Window) -> String {
    let dialog = adwaita::AlertDialog::builder()
        .heading("Save Changes?")
        .body("Your changes will be lost if you don't save them.")
        .build();
    
    dialog.add_response("discard", "Discard");
    dialog.add_response("cancel", "Cancel");
    dialog.add_response("save", "Save");
    
    dialog.set_response_appearance("discard", adwaita::ResponseAppearance::Destructive);
    dialog.set_response_appearance("save", adwaita::ResponseAppearance::Suggested);
    
    dialog.set_default_response(Some("save"));
    dialog.set_close_response("cancel");
    
    dialog
        .choose_future(Some(&parent))
        .await
        .unwrap_or("cancel".to_string())
}
```

**Python Example**:

```python
def show_simple_alert(self, parent):
    """Show a simple alert dialog."""
    dialog = Adw.AlertDialog(
        heading="Delete File?",
        body="This action cannot be undone."
    )
    
    dialog.add_response("cancel", "Cancel")
    dialog.add_response("delete", "Delete")
    
    # Make delete button red/destructive
    dialog.set_response_appearance("delete", Adw.ResponseAppearance.DESTRUCTIVE)
    
    # Set default response (activated by Enter)
    dialog.set_default_response("cancel")
    
    # Set close response (activated by Escape)
    dialog.set_close_response("cancel")
    
    dialog.choose(parent, None, self.on_delete_response)

def on_delete_response(self, dialog, result):
    """Handle delete dialog response."""
    try:
        response = dialog.choose_finish(result)
        if response == "delete":
            print("File deleted")
    except Exception as e:
        print(f"Error: {e}")

def show_alert_with_extra_content(self, parent):
    """Show alert with extra content."""
    checkbox = Gtk.CheckButton(label="Don't ask again")
    
    dialog = Adw.AlertDialog(
        heading="Clear History?",
        body="All browsing history will be removed.",
        extra_child=checkbox
    )
    
    dialog.add_response("cancel", "Cancel")
    dialog.add_response("clear", "Clear")
    dialog.set_response_appearance("clear", Adw.ResponseAppearance.DESTRUCTIVE)
    dialog.set_default_response("cancel")
    dialog.set_close_response("cancel")
    
    # Store checkbox reference for callback
    dialog.checkbox = checkbox
    dialog.choose(parent, None, self.on_clear_response)

def on_clear_response(self, dialog, result):
    """Handle clear history response."""
    try:
        response = dialog.choose_finish(result)
        if response == "clear":
            dont_ask = dialog.checkbox.get_active()
            print(f"Clear history (don't ask: {dont_ask})")
    except Exception as e:
        print(f"Error: {e}")

async def show_async_alert(self, parent):
    """Show alert dialog with async/await."""
    dialog = Adw.AlertDialog(
        heading="Save Changes?",
        body="Your changes will be lost if you don't save them."
    )
    
    dialog.add_response("discard", "Discard")
    dialog.add_response("cancel", "Cancel")
    dialog.add_response("save", "Save")
    
    dialog.set_response_appearance("discard", Adw.ResponseAppearance.DESTRUCTIVE)
    dialog.set_response_appearance("save", Adw.ResponseAppearance.SUGGESTED)
    
    dialog.set_default_response("save")
    dialog.set_close_response("cancel")
    
    # Note: Python async with GTK requires additional setup
    # This is a simplified example
    return await dialog.choose_future(parent)
```

### Message Dialogs

For displaying longer messages or information.

**Rust Example**:

```rust
fn show_message_dialog(parent: &impl IsA<gtk::Window>) {
    let dialog = adwaita::AlertDialog::builder()
        .heading("Welcome to the App")
        .body(
            "This application helps you manage your tasks efficiently.\n\n\
            Features:\n\
            • Create and organize tasks\n\
            • Set reminders\n\
            • Sync across devices\n\n\
            Get started by creating your first task!"
        )
        .build();
    
    dialog.add_response("ok", "Get Started");
    dialog.set_response_appearance("ok", adwaita::ResponseAppearance::Suggested);
    dialog.set_default_response(Some("ok"));
    
    dialog.choose(
        Some(parent),
        None::<&gtk::gio::Cancellable>,
        |_| {
            // User clicked "Get Started"
        },
    );
}
```

**Python Example**:

```python
def show_message_dialog(self, parent):
    """Show a message dialog with information."""
    dialog = Adw.AlertDialog(
        heading="Welcome to the App",
        body="This application helps you manage your tasks efficiently.\n\n"
             "Features:\n"
             "• Create and organize tasks\n"
             "• Set reminders\n"
             "• Sync across devices\n\n"
             "Get started by creating your first task!"
    )
    
    dialog.add_response("ok", "Get Started")
    dialog.set_response_appearance("ok", Adw.ResponseAppearance.SUGGESTED)
    dialog.set_default_response("ok")
    
    dialog.choose(parent, None, lambda *args: None)
```

### File Choosers

**Rust Example**:

```rust
use gtk::gio;

// Open file dialog
fn show_open_file_dialog(parent: &impl IsA<gtk::Window>) {
    let dialog = gtk::FileDialog::new();
    dialog.set_title("Open File");
    
    // Set file filters
    let filter = gtk::FileFilter::new();
    filter.set_name(Some("Text Files"));
    filter.add_mime_type("text/plain");
    filter.add_pattern("*.txt");
    
    let filters = gio::ListStore::new::<gtk::FileFilter>();
    filters.append(&filter);
    dialog.set_filters(Some(&filters));
    
    dialog.open(
        Some(parent),
        None::<&gio::Cancellable>,
        |result| {
            if let Ok(file) = result {
                let path = file.path().unwrap();
                println!("Selected: {:?}", path);
            }
        },
    );
}

// Save file dialog
fn show_save_file_dialog(parent: &impl IsA<gtk::Window>) {
    let dialog = gtk::FileDialog::new();
    dialog.set_title("Save File");
    dialog.set_initial_name(Some("Untitled.txt"));
    
    dialog.save(
        Some(parent),
        None::<&gio::Cancellable>,
        |result| {
            if let Ok(file) = result {
                let path = file.path().unwrap();
                println!("Save to: {:?}", path);
            }
        },
    );
}

// Select folder dialog
fn show_select_folder_dialog(parent: &impl IsA<gtk::Window>) {
    let dialog = gtk::FileDialog::new();
    dialog.set_title("Select Folder");
    
    dialog.select_folder(
        Some(parent),
        None::<&gio::Cancellable>,
        |result| {
            if let Ok(file) = result {
                let path = file.path().unwrap();
                println!("Selected folder: {:?}", path);
            }
        },
    );
}

// Multiple files selection
fn show_open_multiple_files_dialog(parent: &impl IsA<gtk::Window>) {
    let dialog = gtk::FileDialog::new();
    dialog.set_title("Select Files");
    
    dialog.open_multiple(
        Some(parent),
        None::<&gio::Cancellable>,
        |result| {
            if let Ok(files) = result {
                for i in 0..files.n_items() {
                    if let Some(file) = files.item(i).and_downcast::<gio::File>() {
                        println!("Selected: {:?}", file.path());
                    }
                }
            }
        },
    );
}
```

**Python Example**:

```python
def show_open_file_dialog(self, parent):
    """Show open file dialog."""
    dialog = Gtk.FileDialog(title="Open File")
    
    # Set file filters
    filter_text = Gtk.FileFilter()
    filter_text.set_name("Text Files")
    filter_text.add_mime_type("text/plain")
    filter_text.add_pattern("*.txt")
    
    filters = Gio.ListStore.new(Gtk.FileFilter)
    filters.append(filter_text)
    dialog.set_filters(filters)
    
    dialog.open(parent, None, self.on_open_file_response)

def on_open_file_response(self, dialog, result):
    """Handle open file response."""
    try:
        file = dialog.open_finish(result)
        if file:
            path = file.get_path()
            print(f"Selected: {path}")
    except Exception as e:
        print(f"No file selected: {e}")

def show_save_file_dialog(self, parent):
    """Show save file dialog."""
    dialog = Gtk.FileDialog(
        title="Save File",
        initial_name="Untitled.txt"
    )
    
    dialog.save(parent, None, self.on_save_file_response)

def on_save_file_response(self, dialog, result):
    """Handle save file response."""
    try:
        file = dialog.save_finish(result)
        if file:
            path = file.get_path()
            print(f"Save to: {path}")
    except Exception as e:
        print(f"Save cancelled: {e}")

def show_select_folder_dialog(self, parent):
    """Show select folder dialog."""
    dialog = Gtk.FileDialog(title="Select Folder")
    
    dialog.select_folder(parent, None, self.on_folder_selected)

def on_folder_selected(self, dialog, result):
    """Handle folder selection."""
    try:
        file = dialog.select_folder_finish(result)
        if file:
            path = file.get_path()
            print(f"Selected folder: {path}")
    except Exception as e:
        print(f"No folder selected: {e}")

def show_open_multiple_files_dialog(self, parent):
    """Show dialog to select multiple files."""
    dialog = Gtk.FileDialog(title="Select Files")
    
    dialog.open_multiple(parent, None, self.on_multiple_files_selected)

def on_multiple_files_selected(self, dialog, result):
    """Handle multiple files selection."""
    try:
        files = dialog.open_multiple_finish(result)
        if files:
            for i in range(files.get_n_items()):
                file = files.get_item(i)
                print(f"Selected: {file.get_path()}")
    except Exception as e:
        print(f"No files selected: {e}")
```

### Application Menu

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;

fn create_menu_model() -> gio::Menu {
    let menu = gio::Menu::new();
    
    // Add menu items
    menu.append(Some("New Window"), Some("app.new-window"));
    menu.append(Some("Preferences"), Some("app.preferences"));
    
    // Add submenu
    let help_menu = gio::Menu::new();
    help_menu.append(Some("Keyboard Shortcuts"), Some("win.show-help-overlay"));
    help_menu.append(Some("About"), Some("app.about"));
    
    menu.append_submenu(Some("Help"), &help_menu);
    
    menu.append(Some("Quit"), Some("app.quit"));
    
    menu
}

fn setup_menu_button(header: &adwaita::HeaderBar) {
    let menu_model = create_menu_model();
    
    let menu_button = gtk::MenuButton::new();
    menu_button.set_icon_name("open-menu-symbolic");
    menu_button.set_menu_model(Some(&menu_model));
    menu_button.set_tooltip_text(Some("Main Menu"));
    
    header.pack_end(&menu_button);
}

// Primary menu (alternative pattern)
fn create_primary_menu() -> gio::Menu {
    let menu = gio::Menu::new();
    
    // Section 1: New items
    let section1 = gio::Menu::new();
    section1.append(Some("New Tab"), Some("app.new-tab"));
    section1.append(Some("New Window"), Some("app.new-window"));
    menu.append_section(None, &section1);
    
    // Section 2: Settings
    let section2 = gio::Menu::new();
    section2.append(Some("Preferences"), Some("app.preferences"));
    menu.append_section(None, &section2);
    
    // Section 3: Help
    let section3 = gio::Menu::new();
    section3.append(Some("Keyboard Shortcuts"), Some("win.show-help-overlay"));
    section3.append(Some("About"), Some("app.about"));
    menu.append_section(None, &section3);
    
    menu
}
```

**Python Example**:

```python
def create_menu_model(self):
    """Create application menu model."""
    menu = Gio.Menu()
    
    # Add menu items
    menu.append("New Window", "app.new-window")
    menu.append("Preferences", "app.preferences")
    
    # Add submenu
    help_menu = Gio.Menu()
    help_menu.append("Keyboard Shortcuts", "win.show-help-overlay")
    help_menu.append("About", "app.about")
    
    menu.append_submenu("Help", help_menu)
    
    menu.append("Quit", "app.quit")
    
    return menu

def setup_menu_button(self, header):
    """Setup menu button in header bar."""
    menu_model = self.create_menu_model()
    
    menu_button = Gtk.MenuButton()
    menu_button.set_icon_name("open-menu-symbolic")
    menu_button.set_menu_model(menu_model)
    menu_button.set_tooltip_text("Main Menu")
    
    header.pack_end(menu_button)

def create_primary_menu(self):
    """Create primary menu with sections."""
    menu = Gio.Menu()
    
    # Section 1: New items
    section1 = Gio.Menu()
    section1.append("New Tab", "app.new-tab")
    section1.append("New Window", "app.new-window")
    menu.append_section(None, section1)
    
    # Section 2: Settings
    section2 = Gio.Menu()
    section2.append("Preferences", "app.preferences")
    menu.append_section(None, section2)
    
    # Section 3: Help
    section3 = Gio.Menu()
    section3.append("Keyboard Shortcuts", "win.show-help-overlay")
    section3.append("About", "app.about")
    menu.append_section(None, section3)
    
    return menu
```

### Search

**Rust Example**:

```rust
use gtk::prelude::*;

fn create_search_bar() -> (gtk::SearchBar, gtk::SearchEntry) {
    let search_bar = gtk::SearchBar::new();
    let search_entry = gtk::SearchEntry::new();
    
    search_bar.set_child(Some(&search_entry));
    search_bar.set_show_close_button(true);
    search_bar.connect_entry(&search_entry);
    
    (search_bar, search_entry)
}

fn setup_search_in_window(window: &adwaita::ApplicationWindow) {
    let (search_bar, search_entry) = create_search_bar();
    
    // Create toggle button in header
    let search_button = gtk::ToggleButton::new();
    search_button.set_icon_name("system-search-symbolic");
    search_button.set_tooltip_text(Some("Search"));
    
    // Bind button to search bar
    search_button
        .bind_property("active", &search_bar, "search-mode-enabled")
        .flags(glib::BindingFlags::BIDIRECTIONAL | glib::BindingFlags::SYNC_CREATE)
        .build();
    
    // Connect search entry
    search_entry.connect_search_changed(|entry| {
        let text = entry.text();
        println!("Search: {}", text);
        // Perform search here
    });
    
    // Keyboard shortcut
    let search_action = gtk::gio::SimpleAction::new("search", None);
    search_action.connect_activate(glib::clone!(
        #[weak]
        search_bar,
        move |_, _| {
            let mode = search_bar.is_search_mode();
            search_bar.set_search_mode(!mode);
        }
    ));
    window.add_action(&search_action);
}

// Search with popover
fn create_search_popover() -> gtk::Popover {
    let search_entry = gtk::SearchEntry::new();
    search_entry.set_placeholder_text(Some("Search..."));
    
    let list = gtk::ListBox::new();
    list.set_size_request(300, 200);
    
    let vbox = gtk::Box::new(gtk::Orientation::Vertical, 6);
    vbox.set_margin_top(6);
    vbox.set_margin_bottom(6);
    vbox.set_margin_start(6);
    vbox.set_margin_end(6);
    vbox.append(&search_entry);
    vbox.append(&list);
    
    let popover = gtk::Popover::new();
    popover.set_child(Some(&vbox));
    
    search_entry.connect_search_changed(glib::clone!(
        #[weak]
        list,
        move |entry| {
            let query = entry.text();
            update_search_results(&list, &query);
        }
    ));
    
    popover
}

fn update_search_results(list: &gtk::ListBox, query: &str) {
    // Clear existing results
    while let Some(child) = list.first_child() {
        list.remove(&child);
    }
    
    // Add new results
    if !query.is_empty() {
        for i in 1..=5 {
            let label = gtk::Label::new(Some(&format!("Result {} for '{}'", i, query)));
            label.set_xalign(0.0);
            list.append(&label);
        }
    }
}
```

**Python Example**:

```python
def create_search_bar(self):
    """Create search bar with entry."""
    self.search_bar = Gtk.SearchBar()
    self.search_entry = Gtk.SearchEntry()
    
    self.search_bar.set_child(self.search_entry)
    self.search_bar.set_show_close_button(True)
    self.search_bar.connect_entry(self.search_entry)
    
    return self.search_bar

def setup_search_in_window(self, window):
    """Setup search functionality in window."""
    search_bar = self.create_search_bar()
    
    # Create toggle button in header
    search_button = Gtk.ToggleButton()
    search_button.set_icon_name("system-search-symbolic")
    search_button.set_tooltip_text("Search")
    
    # Bind button to search bar
    search_button.bind_property(
        "active", search_bar, "search-mode-enabled",
        GObject.BindingFlags.BIDIRECTIONAL | GObject.BindingFlags.SYNC_CREATE
    )
    
    # Connect search entry
    self.search_entry.connect('search-changed', self.on_search_changed)
    
    # Keyboard shortcut
    search_action = Gio.SimpleAction.new("search", None)
    search_action.connect('activate', self.on_search_activated)
    window.add_action(search_action)

def on_search_changed(self, entry):
    """Handle search text change."""
    text = entry.get_text()
    print(f"Search: {text}")
    # Perform search here

def on_search_activated(self, action, param):
    """Toggle search mode."""
    mode = self.search_bar.get_search_mode()
    self.search_bar.set_search_mode(not mode)

def create_search_popover(self):
    """Create search popover with results."""
    search_entry = Gtk.SearchEntry()
    search_entry.set_placeholder_text("Search...")
    
    self.search_list = Gtk.ListBox()
    self.search_list.set_size_request(300, 200)
    
    vbox = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=6)
    vbox.set_margin_top(6)
    vbox.set_margin_bottom(6)
    vbox.set_margin_start(6)
    vbox.set_margin_end(6)
    vbox.append(search_entry)
    vbox.append(self.search_list)
    
    popover = Gtk.Popover()
    popover.set_child(vbox)
    
    search_entry.connect('search-changed', self.on_popover_search_changed)
    
    return popover

def on_popover_search_changed(self, entry):
    """Handle search in popover."""
    query = entry.get_text()
    self.update_search_results(query)

def update_search_results(self, query):
    """Update search results list."""
    # Clear existing results
    child = self.search_list.get_first_child()
    while child:
        next_child = child.get_next_sibling()
        self.search_list.remove(child)
        child = next_child
    
    # Add new results
    if query:
        for i in range(1, 6):
            label = Gtk.Label(label=f"Result {i} for '{query}'")
            label.set_xalign(0.0)
            self.search_list.append(label)
```

### Keyboard Shortcuts

**Rust Example**:

```rust
use gtk::prelude::*;

fn setup_keyboard_shortcuts(app: &adwaita::Application) {
    // Set accelerators for actions
    app.set_accels_for_action("app.quit", &["<Ctrl>Q"]);
    app.set_accels_for_action("app.preferences", &["<Ctrl>comma"]);
    app.set_accels_for_action("app.about", &["<Ctrl>question"]);
    app.set_accels_for_action("win.close", &["<Ctrl>W"]);
    app.set_accels_for_action("win.search", &["<Ctrl>F"]);
    app.set_accels_for_action("win.new-tab", &["<Ctrl>T"]);
    app.set_accels_for_action("win.close-tab", &["<Ctrl>W"]);
    app.set_accels_for_action("win.next-tab", &["<Ctrl>Page_Down"]);
    app.set_accels_for_action("win.prev-tab", &["<Ctrl>Page_Up"]);
}

// Keyboard shortcuts help overlay
fn create_shortcuts_window(parent: &impl IsA<gtk::Window>) -> gtk::ShortcutsWindow {
    let shortcuts = gtk::ShortcutsWindow::new();
    shortcuts.set_transient_for(Some(parent));
    shortcuts.set_modal(true);
    
    let section = gtk::ShortcutsSection::new();
    section.set_visible(true);
    
    // General group
    let general_group = gtk::ShortcutsGroup::new();
    general_group.set_title("General");
    general_group.set_visible(true);
    
    add_shortcut(&general_group, "Search", "<Ctrl>F");
    add_shortcut(&general_group, "Preferences", "<Ctrl>comma");
    add_shortcut(&general_group, "Quit", "<Ctrl>Q");
    
    section.append(&general_group);
    
    // Tabs group
    let tabs_group = gtk::ShortcutsGroup::new();
    tabs_group.set_title("Tabs");
    tabs_group.set_visible(true);
    
    add_shortcut(&tabs_group, "New Tab", "<Ctrl>T");
    add_shortcut(&tabs_group, "Close Tab", "<Ctrl>W");
    add_shortcut(&tabs_group, "Next Tab", "<Ctrl>Page_Down");
    add_shortcut(&tabs_group, "Previous Tab", "<Ctrl>Page_Up");
    
    section.append(&tabs_group);
    
    shortcuts.add(&section);
    shortcuts
}

fn add_shortcut(group: &gtk::ShortcutsGroup, title: &str, accelerator: &str) {
    let shortcut = gtk::ShortcutsShortcut::new();
    shortcut.set_title(title);
    shortcut.set_accelerator(accelerator);
    shortcut.set_visible(true);
    group.append(&shortcut);
}

// Event controller for custom shortcuts
fn setup_custom_key_handler(window: &impl IsA<gtk::Widget>) {
    let controller = gtk::EventControllerKey::new();
    
    controller.connect_key_pressed(|_, keyval, _, modifier| {
        if modifier == gtk::gdk::ModifierType::CONTROL_MASK {
            match keyval {
                gtk::gdk::Key::s => {
                    println!("Save");
                    return glib::Propagation::Stop;
                }
                gtk::gdk::Key::o => {
                    println!("Open");
                    return glib::Propagation::Stop;
                }
                _ => {}
            }
        }
        glib::Propagation::Proceed
    });
    
    window.add_controller(controller);
}
```

**Python Example**:

```python
def setup_keyboard_shortcuts(self, app):
    """Setup keyboard shortcuts for application."""
    # Set accelerators for actions
    app.set_accels_for_action("app.quit", ["<Ctrl>Q"])
    app.set_accels_for_action("app.preferences", ["<Ctrl>comma"])
    app.set_accels_for_action("app.about", ["<Ctrl>question"])
    app.set_accels_for_action("win.close", ["<Ctrl>W"])
    app.set_accels_for_action("win.search", ["<Ctrl>F"])
    app.set_accels_for_action("win.new-tab", ["<Ctrl>T"])
    app.set_accels_for_action("win.close-tab", ["<Ctrl>W"])
    app.set_accels_for_action("win.next-tab", ["<Ctrl>Page_Down"])
    app.set_accels_for_action("win.prev-tab", ["<Ctrl>Page_Up"])

def create_shortcuts_window(self, parent):
    """Create keyboard shortcuts help overlay."""
    shortcuts = Gtk.ShortcutsWindow()
    shortcuts.set_transient_for(parent)
    shortcuts.set_modal(True)
    
    section = Gtk.ShortcutsSection()
    section.set_visible(True)
    
    # General group
    general_group = Gtk.ShortcutsGroup()
    general_group.set_title("General")
    general_group.set_visible(True)
    
    self.add_shortcut(general_group, "Search", "<Ctrl>F")
    self.add_shortcut(general_group, "Preferences", "<Ctrl>comma")
    self.add_shortcut(general_group, "Quit", "<Ctrl>Q")
    
    section.append(general_group)
    
    # Tabs group
    tabs_group = Gtk.ShortcutsGroup()
    tabs_group.set_title("Tabs")
    tabs_group.set_visible(True)
    
    self.add_shortcut(tabs_group, "New Tab", "<Ctrl>T")
    self.add_shortcut(tabs_group, "Close Tab", "<Ctrl>W")
    self.add_shortcut(tabs_group, "Next Tab", "<Ctrl>Page_Down")
    self.add_shortcut(tabs_group, "Previous Tab", "<Ctrl>Page_Up")
    
    section.append(tabs_group)
    
    shortcuts.add(section)
    return shortcuts

def add_shortcut(self, group, title, accelerator):
    """Add a shortcut to a group."""
    shortcut = Gtk.ShortcutsShortcut()
    shortcut.set_title(title)
    shortcut.set_accelerator(accelerator)
    shortcut.set_visible(True)
    group.append(shortcut)

def setup_custom_key_handler(self, window):
    """Setup custom key handler for window."""
    controller = Gtk.EventControllerKey()
    
    def on_key_pressed(controller, keyval, keycode, state):
        if state & Gdk.ModifierType.CONTROL_MASK:
            if keyval == Gdk.KEY_s:
                print("Save")
                return True
            elif keyval == Gdk.KEY_o:
                print("Open")
                return True
        return False
    
    controller.connect('key-pressed', on_key_pressed)
    window.add_controller(controller)
```



## Chapter 10: System Integration

### D-Bus Communication

D-Bus is the inter-process communication system used in GNOME.

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;
use gtk::prelude::*;

// Send notification via D-Bus
fn send_notification(app_name: &str, summary: &str, body: &str) {
    let connection = gio::bus_get_sync(gio::BusType::Session, None::<&gio::Cancellable>).unwrap();
    
    let message = gio::DBusMessage::new_method_call(
        Some("org.freedesktop.Notifications"),
        "/org/freedesktop/Notifications",
        Some("org.freedesktop.Notifications"),
        "Notify",
    );
    
    // Build parameters
    let app_name_v = glib::Variant::from(app_name);
    let replaces_id_v = glib::Variant::from(0u32);
    let icon_v = glib::Variant::from("");
    let summary_v = glib::Variant::from(summary);
    let body_v = glib::Variant::from(body);
    let actions_v = glib::Variant::array_from_iter::<String>(vec![]);
    let hints_v = glib::Variant::array_from_dict_entries::<String, glib::Variant>(None);
    let timeout_v = glib::Variant::from(-1i32);
    
    let params = glib::Variant::tuple_from_iter(vec![
        app_name_v,
        replaces_id_v,
        icon_v,
        summary_v,
        body_v,
        actions_v,
        hints_v,
        timeout_v,
    ]);
    
    message.set_body(&params);
    
    let _ = connection.send_message(
        &message,
        gio::DBusSendMessageFlags::NONE,
        None,
    );
}

// Listen to D-Bus signals
fn listen_to_dbus_signals() {
    let connection = gio::bus_get_sync(gio::BusType::Session, None::<&gio::Cancellable>()).unwrap();
    
    connection.signal_subscribe(
        Some("org.freedesktop.Notifications"),
        Some("org.freedesktop.Notifications"),
        Some("NotificationClosed"),
        Some("/org/freedesktop/Notifications"),
        None,
        gio::DBusSignalFlags::NONE,
        |_, _, _, _, signal, params| {
            if let Some(id) = params.and_then(|p| p.child_value(0).get::<u32>()) {
                println!("Notification {} closed", id);
            }
        },
    );
}

// Call D-Bus method
async fn call_dbus_method() -> Result<String, glib::Error> {
    let connection = gio::bus_get_future(gio::BusType::Session).await?;
    
    let result = connection
        .call_future(
            Some("org.freedesktop.DBus"),
            "/org/freedesktop/DBus",
            Some("org.freedesktop.DBus"),
            "ListNames",
            None,
            Some(&glib::VariantTy::new("(as)").unwrap()),
            gio::DBusCallFlags::NONE,
            -1,
        )
        .await?;
    
    if let Some(names) = result.child_value(0).get::<Vec<String>>() {
        Ok(format!("Found {} services", names.len()))
    } else {
        Ok("No services found".to_string())
    }
}
```

**Python Example**:

```python
def send_notification(self, app_name, summary, body):
    """Send notification via D-Bus."""
    try:
        connection = Gio.bus_get_sync(Gio.BusType.SESSION, None)
        
        message = Gio.DBusMessage.new_method_call(
            "org.freedesktop.Notifications",
            "/org/freedesktop/Notifications",
            "org.freedesktop.Notifications",
            "Notify"
        )
        
        # Build parameters
        params = GLib.Variant('(susssasa{sv}i)', (
            app_name,      # app_name
            0,             # replaces_id
            "",            # icon
            summary,       # summary
            body,          # body
            [],            # actions
            {},            # hints
            -1             # timeout
        ))
        
        message.set_body(params)
        
        connection.send_message(
            message,
            Gio.DBusSendMessageFlags.NONE,
            None
        )
    except Exception as e:
        print(f"Failed to send notification: {e}")

def listen_to_dbus_signals(self):
    """Listen to D-Bus signals."""
    try:
        connection = Gio.bus_get_sync(Gio.BusType.SESSION, None)
        
        connection.signal_subscribe(
            "org.freedesktop.Notifications",
            "org.freedesktop.Notifications",
            "NotificationClosed",
            "/org/freedesktop/Notifications",
            None,
            Gio.DBusSignalFlags.NONE,
            self.on_notification_closed,
            None
        )
    except Exception as e:
        print(f"Failed to subscribe to signals: {e}")

def on_notification_closed(self, connection, sender, object_path, 
                           interface, signal, params, user_data):
    """Handle notification closed signal."""
    if params:
        notification_id = params[0]
        print(f"Notification {notification_id} closed")

async def call_dbus_method(self):
    """Call D-Bus method asynchronously."""
    try:
        connection = await Gio.bus_get_async(Gio.BusType.SESSION, None)
        
        result = await connection.call_async(
            "org.freedesktop.DBus",
            "/org/freedesktop/DBus",
            "org.freedesktop.DBus",
            "ListNames",
            None,
            GLib.VariantType.new("(as)"),
            Gio.DBusCallFlags.NONE,
            -1,
            None
        )
        
        names = result[0]
        return f"Found {len(names)} services"
    except Exception as e:
        return f"Error: {e}"
```

### Notifications

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;

fn send_simple_notification(app: &gio::Application, title: &str, body: &str) {
    let notification = gio::Notification::new(title);
    notification.set_body(Some(body));
    app.send_notification(None, &notification);
}

fn send_notification_with_action(app: &gio::Application) {
    let notification = gio::Notification::new("New Message");
    notification.set_body(Some("You have a new message from Alice"));
    notification.set_icon(&gio::ThemedIcon::new("mail-unread-symbolic"));
    
    // Add action
    notification.add_button("Reply", "app.reply");
    notification.set_default_action("app.open-message");
    
    // Set priority
    notification.set_priority(gio::NotificationPriority::High);
    
    app.send_notification(Some("message-1"), &notification);
}

fn withdraw_notification(app: &gio::Application, id: &str) {
    app.withdraw_notification(id);
}
```

**Python Example**:

```python
def send_simple_notification(self, app, title, body):
    """Send a simple notification."""
    notification = Gio.Notification.new(title)
    notification.set_body(body)
    app.send_notification(None, notification)

def send_notification_with_action(self, app):
    """Send notification with actions."""
    notification = Gio.Notification.new("New Message")
    notification.set_body("You have a new message from Alice")
    notification.set_icon(Gio.ThemedIcon.new("mail-unread-symbolic"))
    
    # Add action
    notification.add_button("Reply", "app.reply")
    notification.set_default_action("app.open-message")
    
    # Set priority
    notification.set_priority(Gio.NotificationPriority.HIGH)
    
    app.send_notification("message-1", notification)

def withdraw_notification(self, app, notification_id):
    """Withdraw a notification."""
    app.withdraw_notification(notification_id)
```

### Portal Integration

GNOME applications use portals for system integration in sandboxed environments (Flatpak).

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;
use gtk::prelude::*;

// Open URI with portal
fn open_uri_with_portal(parent: &impl IsA<gtk::Window>, uri: &str) {
    gtk::UriLauncher::new(uri).launch(
        Some(parent),
        None::<&gio::Cancellable>,
        |result| {
            match result {
                Ok(_) => println!("URI opened successfully"),
                Err(e) => println!("Failed to open URI: {}", e),
            }
        },
    );
}

// Open file with default application
fn open_file_with_portal(parent: &impl IsA<gtk::Window>, file: &gio::File) {
    gtk::FileLauncher::new(Some(file)).open(
        Some(parent),
        None::<&gio::Cancellable>,
        |result| {
            match result {
                Ok(_) => println!("File opened successfully"),
                Err(e) => println!("Failed to open file: {}", e),
            }
        },
    );
}

// Take screenshot (portal)
async fn take_screenshot(parent: &gtk::Window) -> Result<String, glib::Error> {
    let connection = gio::bus_get_future(gio::BusType::Session).await?;
    
    // Call screenshot portal
    let result = connection
        .call_future(
            Some("org.freedesktop.portal.Desktop"),
            "/org/freedesktop/portal/desktop",
            Some("org.freedesktop.portal.Screenshot"),
            "Screenshot",
            Some(&glib::Variant::tuple_from_iter(vec![
                glib::Variant::from(""),  // parent_window
                glib::Variant::from_dict_entry(
                    glib::Variant::from("modal"),
                    glib::Variant::from(&glib::Variant::from(true)),
                ),
            ])),
            Some(&glib::VariantTy::new("(o)").unwrap()),
            gio::DBusCallFlags::NONE,
            -1,
        )
        .await?;
    
    Ok("Screenshot taken".to_string())
}

// Access camera (portal)
fn request_camera_access(parent: &impl IsA<gtk::Window>) {
    // This would typically use the Camera portal
    // Implementation depends on specific portal API
    println!("Requesting camera access via portal");
}

// Request location access
fn request_location_access() {
    // Use Geoclue or Location portal
    println!("Requesting location access");
}
```

**Python Example**:

```python
def open_uri_with_portal(self, parent, uri):
    """Open URI using portal."""
    launcher = Gtk.UriLauncher.new(uri)
    launcher.launch(parent, None, self.on_uri_opened)

def on_uri_opened(self, launcher, result):
    """Handle URI opened result."""
    try:
        launcher.launch_finish(result)
        print("URI opened successfully")
    except Exception as e:
        print(f"Failed to open URI: {e}")

def open_file_with_portal(self, parent, file):
    """Open file with default application."""
    launcher = Gtk.FileLauncher.new(file)
    launcher.open(parent, None, self.on_file_opened)

def on_file_opened(self, launcher, result):
    """Handle file opened result."""
    try:
        launcher.open_finish(result)
        print("File opened successfully")
    except Exception as e:
        print(f"Failed to open file: {e}")

async def take_screenshot(self, parent):
    """Take screenshot using portal."""
    try:
        connection = await Gio.bus_get_async(Gio.BusType.SESSION, None)
        
        # Call screenshot portal
        result = await connection.call_async(
            "org.freedesktop.portal.Desktop",
            "/org/freedesktop/portal/desktop",
            "org.freedesktop.portal.Screenshot",
            "Screenshot",
            GLib.Variant('(sa{sv})', ("", {"modal": GLib.Variant('b', True)})),
            GLib.VariantType.new("(o)"),
            Gio.DBusCallFlags.NONE,
            -1,
            None
        )
        
        return "Screenshot taken"
    except Exception as e:
        return f"Error: {e}"

def request_camera_access(self, parent):
    """Request camera access via portal."""
    # Implementation depends on specific portal API
    print("Requesting camera access via portal")

def request_location_access(self):
    """Request location access."""
    # Use Geoclue or Location portal
    print("Requesting location access")
```

### Background Tasks

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;
use std::cell::RefCell;
use std::rc::Rc;

// Run task in background thread
fn run_background_task<F, R>(task: F) -> glib::JoinHandle<R>
where
    F: FnOnce() -> R + Send + 'static,
    R: Send + 'static,
{
    gio::spawn_blocking(task)
}

// Example: Load file in background
fn load_file_async(path: String, callback: impl FnOnce(Result<String, String>) + 'static) {
    gio::spawn_blocking(move || {
        std::fs::read_to_string(&path)
            .map_err(|e| e.to_string())
    })
    .then(move |result| {
        callback(result);
    });
}

// Progress tracking for long operations
struct TaskProgress {
    progress: Rc<RefCell<f64>>,
    cancelled: Rc<RefCell<bool>>,
}

impl TaskProgress {
    fn new() -> Self {
        Self {
            progress: Rc::new(RefCell::new(0.0)),
            cancelled: Rc::new(RefCell::new(false)),
        }
    }
    
    fn set_progress(&self, value: f64) {
        *self.progress.borrow_mut() = value;
    }
    
    fn get_progress(&self) -> f64 {
        *self.progress.borrow()
    }
    
    fn cancel(&self) {
        *self.cancelled.borrow_mut() = true;
    }
    
    fn is_cancelled(&self) -> bool {
        *self.cancelled.borrow()
    }
}

fn perform_long_task(progress: TaskProgress) {
    gio::spawn_blocking(move || {
        for i in 0..100 {
            if progress.is_cancelled() {
                return Err("Cancelled");
            }
            
            // Simulate work
            std::thread::sleep(std::time::Duration::from_millis(50));
            
            // Update progress
            let percent = (i as f64) / 100.0;
            glib::idle_add_once(glib::clone!(
                #[weak]
                progress,
                move || {
                    progress.set_progress(percent);
                }
            ));
        }
        Ok("Complete")
    });
}

// Async with futures
async fn async_task_example() -> Result<String, String> {
    // Simulate async work
    glib::timeout_future(std::time::Duration::from_secs(2)).await;
    Ok("Task completed".to_string())
}

fn run_async_task(
    callback: impl FnOnce(Result<String, String>) + 'static
) {
    glib::spawn_future_local(async move {
        let result = async_task_example().await;
        callback(result);
    });
}
```

**Python Example**:

```python
import threading
from gi.repository import GLib

def run_background_task(self, task, callback):
    """Run task in background thread."""
    def worker():
        result = task()
        GLib.idle_add(callback, result)
    
    thread = threading.Thread(target=worker)
    thread.daemon = True
    thread.start()

def load_file_async(self, path, callback):
    """Load file asynchronously."""
    def task():
        try:
            with open(path, 'r') as f:
                return f.read()
        except Exception as e:
            return str(e)
    
    self.run_background_task(task, callback)

class TaskProgress:
    """Track progress of long-running tasks."""
    
    def __init__(self):
        self.progress = 0.0
        self.cancelled = False
        self._lock = threading.Lock()
    
    def set_progress(self, value):
        """Set progress value (0.0 to 1.0)."""
        with self._lock:
            self.progress = value
    
    def get_progress(self):
        """Get current progress."""
        with self._lock:
            return self.progress
    
    def cancel(self):
        """Cancel the task."""
        with self._lock:
            self.cancelled = True
    
    def is_cancelled(self):
        """Check if task is cancelled."""
        with self._lock:
            return self.cancelled

def perform_long_task(self, progress_tracker, callback):
    """Perform a long-running task with progress tracking."""
    def task():
        import time
        for i in range(100):
            if progress_tracker.is_cancelled():
                return "Cancelled"
            
            # Simulate work
            time.sleep(0.05)
            
            # Update progress on main thread
            percent = i / 100.0
            GLib.idle_add(progress_tracker.set_progress, percent)
        
        return "Complete"
    
    self.run_background_task(task, callback)

async def async_task_example(self):
    """Example async task."""
    import asyncio
    await asyncio.sleep(2)
    return "Task completed"

def run_async_task(self, callback):
    """Run async task."""
    import asyncio
    
    async def wrapper():
        result = await self.async_task_example()
        GLib.idle_add(callback, result)
    
    # Note: Requires asyncio integration with GLib main loop
    asyncio.create_task(wrapper())
```

### System Settings Integration

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;
use gtk::prelude::*;

// Read system color scheme
fn get_color_scheme() -> adwaita::ColorScheme {
    let settings = adwaita::StyleManager::default();
    settings.color_scheme()
}

// Listen to color scheme changes
fn watch_color_scheme_changes<F>(callback: F)
where
    F: Fn(adwaita::ColorScheme) + 'static,
{
    let style_manager = adwaita::StyleManager::default();
    style_manager.connect_color_scheme_notify(move |manager| {
        let scheme = manager.color_scheme();
        callback(scheme);
    });
}

// Force dark mode
fn set_dark_mode(enabled: bool) {
    let style_manager = adwaita::StyleManager::default();
    if enabled {
        style_manager.set_color_scheme(adwaita::ColorScheme::ForceDark);
    } else {
        style_manager.set_color_scheme(adwaita::ColorScheme::Default);
    }
}

// Get system font
fn get_system_font() -> Option<String> {
    let settings = gio::Settings::new("org.gnome.desktop.interface");
    settings.string("font-name").as_str().to_owned().into()
}

// Get text scaling factor
fn get_text_scale_factor() -> f64 {
    let settings = gio::Settings::new("org.gnome.desktop.interface");
    settings.double("text-scaling-factor")
}

// Listen to system settings
fn watch_system_settings<F>(callback: F)
where
    F: Fn() + 'static,
{
    let settings = gio::Settings::new("org.gnome.desktop.interface");
    settings.connect_changed(None, move |_, _| {
        callback();
    });
}

// Check if running under Wayland
fn is_wayland() -> bool {
    std::env::var("WAYLAND_DISPLAY").is_ok()
}

// Check if running under X11
fn is_x11() -> bool {
    std::env::var("DISPLAY").is_ok() && !is_wayland()
}

// Get session type
fn get_session_type() -> String {
    std::env::var("XDG_SESSION_TYPE").unwrap_or_else(|_| "unknown".to_string())
}
```

**Python Example**:

```python
import os

def get_color_scheme(self):
    """Get current system color scheme."""
    style_manager = Adw.StyleManager.get_default()
    return style_manager.get_color_scheme()

def watch_color_scheme_changes(self, callback):
    """Listen to color scheme changes."""
    style_manager = Adw.StyleManager.get_default()
    style_manager.connect('notify::color-scheme', lambda *args: callback())

def set_dark_mode(self, enabled):
    """Force dark or light mode."""
    style_manager = Adw.StyleManager.get_default()
    if enabled:
        style_manager.set_color_scheme(Adw.ColorScheme.FORCE_DARK)
    else:
        style_manager.set_color_scheme(Adw.ColorScheme.DEFAULT)

def get_system_font(self):
    """Get system font setting."""
    settings = Gio.Settings.new("org.gnome.desktop.interface")
    return settings.get_string("font-name")

def get_text_scale_factor(self):
    """Get text scaling factor."""
    settings = Gio.Settings.new("org.gnome.desktop.interface")
    return settings.get_double("text-scaling-factor")

def watch_system_settings(self, callback):
    """Listen to system settings changes."""
    settings = Gio.Settings.new("org.gnome.desktop.interface")
    settings.connect('changed', lambda *args: callback())

def is_wayland(self):
    """Check if running under Wayland."""
    return 'WAYLAND_DISPLAY' in os.environ

def is_x11(self):
    """Check if running under X11."""
    return 'DISPLAY' in os.environ and not self.is_wayland()

def get_session_type(self):
    """Get session type."""
    return os.environ.get('XDG_SESSION_TYPE', 'unknown')
```

### Custom URI Schemes

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;

// Register custom URI scheme handler
fn register_uri_scheme_handler(app: &gio::Application) {
    app.connect_open(|app, files, hint| {
        for file in files {
            if let Some(uri) = file.uri().as_str().strip_prefix("myapp://") {
                handle_custom_uri(app, uri);
            }
        }
    });
}

fn handle_custom_uri(app: &gio::Application, uri: &str) {
    println!("Handling custom URI: {}", uri);
    
    // Parse URI
    let parts: Vec<&str> = uri.split('/').collect();
    match parts.get(0) {
        Some(&"open") => {
            if let Some(id) = parts.get(1) {
                println!("Opening item: {}", id);
                // Open specific item
            }
        }
        Some(&"action") => {
            if let Some(action) = parts.get(1) {
                println!("Performing action: {}", action);
                // Perform action
            }
        }
        _ => {
            println!("Unknown URI scheme");
        }
    }
}

// Desktop file entry for URI scheme:
// Add to .desktop file:
// MimeType=x-scheme-handler/myapp;
```

**Python Example**:

```python
def register_uri_scheme_handler(self, app):
    """Register custom URI scheme handler."""
    app.connect('open', self.on_open_files)

def on_open_files(self, app, files, n_files, hint):
    """Handle file open (including custom URIs)."""
    for file in files:
        uri = file.get_uri()
        if uri.startswith("myapp://"):
            custom_uri = uri[8:]  # Remove "myapp://"
            self.handle_custom_uri(app, custom_uri)

def handle_custom_uri(self, app, uri):
    """Handle custom URI."""
    print(f"Handling custom URI: {uri}")
    
    # Parse URI
    parts = uri.split('/')
    if len(parts) > 0:
        if parts[0] == "open" and len(parts) > 1:
            item_id = parts[1]
            print(f"Opening item: {item_id}")
            # Open specific item
        elif parts[0] == "action" and len(parts) > 1:
            action = parts[1]
            print(f"Performing action: {action}")
            # Perform action
        else:
            print("Unknown URI scheme")

# Desktop file entry for URI scheme:
# Add to .desktop file:
# MimeType=x-scheme-handler/myapp;
```



## Chapter 11: Data Management

### GSettings (Preferences)

GSettings is the standard way to store application preferences.

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;
use gtk::prelude::*;

// Define settings keys
struct AppSettings {
    settings: gio::Settings,
}

impl AppSettings {
    fn new(app_id: &str) -> Self {
        Self {
            settings: gio::Settings::new(app_id),
        }
    }
    
    // Boolean settings
    fn get_dark_mode(&self) -> bool {
        self.settings.boolean("dark-mode")
    }
    
    fn set_dark_mode(&self, value: bool) {
        self.settings.set_boolean("dark-mode", value).ok();
    }
    
    // String settings
    fn get_theme(&self) -> String {
        self.settings.string("theme").to_string()
    }
    
    fn set_theme(&self, value: &str) {
        self.settings.set_string("theme", value).ok();
    }
    
    // Integer settings
    fn get_window_width(&self) -> i32 {
        self.settings.int("window-width")
    }
    
    fn set_window_width(&self, value: i32) {
        self.settings.set_int("window-width", value).ok();
    }
    
    // Array settings
    fn get_recent_files(&self) -> Vec<String> {
        self.settings
            .strv("recent-files")
            .iter()
            .map(|s| s.to_string())
            .collect()
    }
    
    fn set_recent_files(&self, files: &[String]) {
        let strs: Vec<&str> = files.iter().map(|s| s.as_str()).collect();
        self.settings.set_strv("recent-files", &strs).ok();
    }
    
    // Enum settings
    fn get_sort_order(&self) -> String {
        self.settings.enum_("sort-order").to_string()
    }
    
    fn set_sort_order(&self, value: &str) {
        self.settings.set_enum("sort-order", value).ok();
    }
    
    // Listen to changes
    fn connect_changed<F>(&self, key: Option<&str>, callback: F)
    where
        F: Fn(&str) + 'static,
    {
        self.settings.connect_changed(key, move |_, key| {
            callback(key);
        });
    }
    
    // Reset to defaults
    fn reset(&self, key: &str) {
        self.settings.reset(key);
    }
}

// Bind setting to widget
fn bind_setting_to_widget(settings: &gio::Settings, widget: &adwaita::SwitchRow) {
    settings
        .bind("dark-mode", widget, "active")
        .build();
}

// Settings with custom change handler
fn setup_settings_with_handler(settings: &gio::Settings) {
    settings.connect_changed(Some("theme"), |settings, key| {
        let theme = settings.string(key);
        println!("Theme changed to: {}", theme);
        // Apply theme
    });
}
```

**Python Example**:

```python
class AppSettings:
    """Application settings manager."""
    
    def __init__(self, app_id):
        self.settings = Gio.Settings.new(app_id)
    
    # Boolean settings
    def get_dark_mode(self):
        """Get dark mode preference."""
        return self.settings.get_boolean('dark-mode')
    
    def set_dark_mode(self, value):
        """Set dark mode preference."""
        self.settings.set_boolean('dark-mode', value)
    
    # String settings
    def get_theme(self):
        """Get theme preference."""
        return self.settings.get_string('theme')
    
    def set_theme(self, value):
        """Set theme preference."""
        self.settings.set_string('theme', value)
    
    # Integer settings
    def get_window_width(self):
        """Get window width."""
        return self.settings.get_int('window-width')
    
    def set_window_width(self, value):
        """Set window width."""
        self.settings.set_int('window-width', value)
    
    # Array settings
    def get_recent_files(self):
        """Get recent files list."""
        return self.settings.get_strv('recent-files')
    
    def set_recent_files(self, files):
        """Set recent files list."""
        self.settings.set_strv('recent-files', files)
    
    # Enum settings
    def get_sort_order(self):
        """Get sort order."""
        return self.settings.get_enum('sort-order')
    
    def set_sort_order(self, value):
        """Set sort order."""
        self.settings.set_enum('sort-order', value)
    
    # Listen to changes
    def connect_changed(self, key, callback):
        """Connect to setting change signal."""
        self.settings.connect(f'changed::{key}', 
                            lambda settings, key: callback(key))
    
    # Reset to defaults
    def reset(self, key):
        """Reset setting to default."""
        self.settings.reset(key)

def bind_setting_to_widget(self, settings, widget):
    """Bind setting to widget."""
    settings.bind('dark-mode', widget, 'active',
                  Gio.SettingsBindFlags.DEFAULT)

def setup_settings_with_handler(self, settings):
    """Setup settings with change handler."""
    def on_theme_changed(settings, key):
        theme = settings.get_string(key)
        print(f"Theme changed to: {theme}")
        # Apply theme
    
    settings.connect('changed::theme', on_theme_changed)
```

#### GSettings Schema Definition

**com.example.MyApp.gschema.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<schemalist>
  <schema id="com.example.MyApp" path="/com/example/MyApp/">
    <!-- Boolean setting -->
    <key name="dark-mode" type="b">
      <default>false</default>
      <summary>Dark mode</summary>
      <description>Use dark color scheme</description>
    </key>
    
    <!-- String setting -->
    <key name="theme" type="s">
      <default>"default"</default>
      <summary>Theme</summary>
      <description>Application theme</description>
    </key>
    
    <!-- Integer setting -->
    <key name="window-width" type="i">
      <default>800</default>
      <summary>Window width</summary>
      <description>The width of the main window</description>
    </key>
    
    <key name="window-height" type="i">
      <default>600</default>
      <summary>Window height</summary>
      <description>The height of the main window</description>
    </key>
    
    <!-- Array setting -->
    <key name="recent-files" type="as">
      <default>[]</default>
      <summary>Recent files</summary>
      <description>List of recently opened files</description>
    </key>
    
    <!-- Enum setting -->
    <key name="sort-order" enum="com.example.MyApp.SortOrder">
      <default>'name'</default>
      <summary>Sort order</summary>
      <description>How to sort items</description>
    </key>
    
    <!-- Double setting -->
    <key name="zoom-level" type="d">
      <default>1.0</default>
      <summary>Zoom level</summary>
      <description>Current zoom level</description>
      <range min="0.5" max="3.0"/>
    </key>
  </schema>
  
  <!-- Enum definition -->
  <enum id="com.example.MyApp.SortOrder">
    <value nick="name" value="0"/>
    <value nick="date" value="1"/>
    <value nick="size" value="2"/>
  </enum>
</schemalist>
```

### JSON Configuration Files

**Rust Example**:

```rust
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::PathBuf;

#[derive(Debug, Serialize, Deserialize, Default)]
struct AppConfig {
    version: String,
    window: WindowConfig,
    editor: EditorConfig,
}

#[derive(Debug, Serialize, Deserialize)]
struct WindowConfig {
    width: i32,
    height: i32,
    maximized: bool,
}

impl Default for WindowConfig {
    fn default() -> Self {
        Self {
            width: 800,
            height: 600,
            maximized: false,
        }
    }
}

#[derive(Debug, Serialize, Deserialize)]
struct EditorConfig {
    font_size: i32,
    show_line_numbers: bool,
    theme: String,
}

impl Default for EditorConfig {
    fn default() -> Self {
        Self {
            font_size: 12,
            show_line_numbers: true,
            theme: "default".to_string(),
        }
    }
}

impl AppConfig {
    fn config_path() -> PathBuf {
        let mut path = glib::user_config_dir();
        path.push("my-app");
        path.push("config.json");
        path
    }
    
    fn load() -> Result<Self, String> {
        let path = Self::config_path();
        
        if !path.exists() {
            return Ok(Self::default());
        }
        
        let content = fs::read_to_string(&path)
            .map_err(|e| format!("Failed to read config: {}", e))?;
        
        serde_json::from_str(&content)
            .map_err(|e| format!("Failed to parse config: {}", e))
    }
    
    fn save(&self) -> Result<(), String> {
        let path = Self::config_path();
        
        // Ensure directory exists
        if let Some(parent) = path.parent() {
            fs::create_dir_all(parent)
                .map_err(|e| format!("Failed to create config dir: {}", e))?;
        }
        
        let content = serde_json::to_string_pretty(self)
            .map_err(|e| format!("Failed to serialize config: {}", e))?;
        
        fs::write(&path, content)
            .map_err(|e| format!("Failed to write config: {}", e))
    }
}

// Usage
fn config_example() {
    // Load config
    let mut config = AppConfig::load().unwrap_or_default();
    
    // Modify
    config.window.width = 1024;
    config.editor.font_size = 14;
    
    // Save
    config.save().ok();
}
```

**Python Example**:

```python
import json
import os
from pathlib import Path
from dataclasses import dataclass, asdict

@dataclass
class WindowConfig:
    """Window configuration."""
    width: int = 800
    height: int = 600
    maximized: bool = False

@dataclass
class EditorConfig:
    """Editor configuration."""
    font_size: int = 12
    show_line_numbers: bool = True
    theme: str = "default"

@dataclass
class AppConfig:
    """Application configuration."""
    version: str = "1.0"
    window: WindowConfig = None
    editor: EditorConfig = None
    
    def __post_init__(self):
        if self.window is None:
            self.window = WindowConfig()
        if self.editor is None:
            self.editor = EditorConfig()
    
    @staticmethod
    def config_path():
        """Get configuration file path."""
        config_dir = Path(GLib.get_user_config_dir()) / "my-app"
        return config_dir / "config.json"
    
    @classmethod
    def load(cls):
        """Load configuration from file."""
        path = cls.config_path()
        
        if not path.exists():
            return cls()
        
        try:
            with open(path, 'r') as f:
                data = json.load(f)
            
            # Convert nested dicts to dataclasses
            if 'window' in data:
                data['window'] = WindowConfig(**data['window'])
            if 'editor' in data:
                data['editor'] = EditorConfig(**data['editor'])
            
            return cls(**data)
        except Exception as e:
            print(f"Failed to load config: {e}")
            return cls()
    
    def save(self):
        """Save configuration to file."""
        path = self.config_path()
        
        # Ensure directory exists
        path.parent.mkdir(parents=True, exist_ok=True)
        
        try:
            # Convert to dict
            data = asdict(self)
            
            with open(path, 'w') as f:
                json.dump(data, f, indent=2)
        except Exception as e:
            print(f"Failed to save config: {e}")

# Usage
def config_example():
    """Configuration usage example."""
    # Load config
    config = AppConfig.load()
    
    # Modify
    config.window.width = 1024
    config.editor.font_size = 14
    
    # Save
    config.save()
```

### SQLite Database

**Rust Example**:

```rust
use rusqlite::{Connection, Result};
use std::path::PathBuf;

struct Database {
    conn: Connection,
}

impl Database {
    fn new() -> Result<Self> {
        let mut path = glib::user_data_dir();
        path.push("my-app");
        std::fs::create_dir_all(&path).ok();
        path.push("database.db");
        
        let conn = Connection::open(path)?;
        
        let db = Self { conn };
        db.create_tables()?;
        Ok(db)
    }
    
    fn create_tables(&self) -> Result<()> {
        self.conn.execute(
            "CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY,
                title TEXT NOT NULL,
                description TEXT,
                completed BOOLEAN NOT NULL DEFAULT 0,
                created_at INTEGER NOT NULL
            )",
            [],
        )?;
        Ok(())
    }
    fn add_task(&self, title: &str, description: Option<&str>) -> Result<i64> {
        let now = std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_secs() as i64;
        
        self.conn.execute(
            "INSERT INTO tasks (title, description, completed, created_at) 
             VALUES (?1, ?2, 0, ?3)",
            (title, description, now),
        )?;
        
        Ok(self.conn.last_insert_rowid())
    }
    
    fn get_task(&self, id: i64) -> Result<Task> {
        let mut stmt = self.conn.prepare(
            "SELECT id, title, description, completed, created_at 
             FROM tasks WHERE id = ?1"
        )?;
        
        stmt.query_row([id], |row| {
            Ok(Task {
                id: row.get(0)?,
                title: row.get(1)?,
                description: row.get(2)?,
                completed: row.get(3)?,
                created_at: row.get(4)?,
            })
        })
    }
    
    fn get_all_tasks(&self) -> Result<Vec<Task>> {
        let mut stmt = self.conn.prepare(
            "SELECT id, title, description, completed, created_at 
             FROM tasks ORDER BY created_at DESC"
        )?;
        
        let tasks = stmt.query_map([], |row| {
            Ok(Task {
                id: row.get(0)?,
                title: row.get(1)?,
                description: row.get(2)?,
                completed: row.get(3)?,
                created_at: row.get(4)?,
            })
        })?;
        
        tasks.collect()
    }
    
    fn update_task(&self, id: i64, title: &str, description: Option<&str>, completed: bool) -> Result<()> {
        self.conn.execute(
            "UPDATE tasks SET title = ?1, description = ?2, completed = ?3 
             WHERE id = ?4",
            (title, description, completed, id),
        )?;
        Ok(())
    }
    
    fn delete_task(&self, id: i64) -> Result<()> {
        self.conn.execute("DELETE FROM tasks WHERE id = ?1", [id])?;
        Ok(())
    }
    
    fn search_tasks(&self, query: &str) -> Result<Vec<Task>> {
        let mut stmt = self.conn.prepare(
            "SELECT id, title, description, completed, created_at 
             FROM tasks 
             WHERE title LIKE ?1 OR description LIKE ?1
             ORDER BY created_at DESC"
        )?;
        
        let search_pattern = format!("%{}%", query);
        let tasks = stmt.query_map([search_pattern], |row| {
            Ok(Task {
                id: row.get(0)?,
                title: row.get(1)?,
                description: row.get(2)?,
                completed: row.get(3)?,
                created_at: row.get(4)?,
            })
        })?;
        
        tasks.collect()
    }
}

#[derive(Debug, Clone)]
struct Task {
    id: i64,
    title: String,
    description: Option<String>,
    completed: bool,
    created_at: i64,
}

// Usage
fn database_example() {
    let db = Database::new().unwrap();
    
    // Add task
    let id = db.add_task("Buy groceries", Some("Milk, eggs, bread")).unwrap();
    
    // Get task
    let task = db.get_task(id).unwrap();
    println!("{:?}", task);
    
    // Update task
    db.update_task(id, "Buy groceries", Some("Milk, eggs, bread, cheese"), true).unwrap();
    
    // Get all tasks
    let tasks = db.get_all_tasks().unwrap();
    for task in tasks {
        println!("{}: {}", task.id, task.title);
    }
    
    // Search tasks
    let results = db.search_tasks("groceries").unwrap();
    
    // Delete task
    db.delete_task(id).unwrap();
}
```

**Python Example**:

```python
import sqlite3
from pathlib import Path
from dataclasses import dataclass
from typing import Optional, List
import time

@dataclass
class Task:
    """Task data class."""
    id: int
    title: str
    description: Optional[str]
    completed: bool
    created_at: int

class Database:
    """SQLite database manager."""
    
    def __init__(self):
        """Initialize database connection."""
        data_dir = Path(GLib.get_user_data_dir()) / "my-app"
        data_dir.mkdir(parents=True, exist_ok=True)
        
        db_path = data_dir / "database.db"
        self.conn = sqlite3.connect(str(db_path))
        self.conn.row_factory = sqlite3.Row
        self.create_tables()
    
    def create_tables(self):
        """Create database tables."""
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY,
                title TEXT NOT NULL,
                description TEXT,
                completed BOOLEAN NOT NULL DEFAULT 0,
                created_at INTEGER NOT NULL
            )
        """)
        self.conn.commit()
    
    def add_task(self, title: str, description: Optional[str] = None) -> int:
        """Add a new task."""
        now = int(time.time())
        cursor = self.conn.execute(
            "INSERT INTO tasks (title, description, completed, created_at) "
            "VALUES (?, ?, 0, ?)",
            (title, description, now)
        )
        self.conn.commit()
        return cursor.lastrowid
    
    def get_task(self, task_id: int) -> Optional[Task]:
        """Get a task by ID."""
        cursor = self.conn.execute(
            "SELECT id, title, description, completed, created_at "
            "FROM tasks WHERE id = ?",
            (task_id,)
        )
        row = cursor.fetchone()
        if row:
            return Task(**dict(row))
        return None
    
    def get_all_tasks(self) -> List[Task]:
        """Get all tasks."""
        cursor = self.conn.execute(
            "SELECT id, title, description, completed, created_at "
            "FROM tasks ORDER BY created_at DESC"
        )
        return [Task(**dict(row)) for row in cursor.fetchall()]
    
    def update_task(self, task_id: int, title: str, 
                   description: Optional[str], completed: bool):
        """Update a task."""
        self.conn.execute(
            "UPDATE tasks SET title = ?, description = ?, completed = ? "
            "WHERE id = ?",
            (title, description, completed, task_id)
        )
        self.conn.commit()
    
    def delete_task(self, task_id: int):
        """Delete a task."""
        self.conn.execute("DELETE FROM tasks WHERE id = ?", (task_id,))
        self.conn.commit()
    
    def search_tasks(self, query: str) -> List[Task]:
        """Search tasks by title or description."""
        search_pattern = f"%{query}%"
        cursor = self.conn.execute(
            "SELECT id, title, description, completed, created_at "
            "FROM tasks "
            "WHERE title LIKE ? OR description LIKE ? "
            "ORDER BY created_at DESC",
            (search_pattern, search_pattern)
        )
        return [Task(**dict(row)) for row in cursor.fetchall()]
    
    def close(self):
        """Close database connection."""
        self.conn.close()

# Usage
def database_example():
    """Database usage example."""
    db = Database()
    
    # Add task
    task_id = db.add_task("Buy groceries", "Milk, eggs, bread")
    
    # Get task
    task = db.get_task(task_id)
    print(task)
    
    # Update task
    db.update_task(task_id, "Buy groceries", 
                   "Milk, eggs, bread, cheese", True)
    
    # Get all tasks
    tasks = db.get_all_tasks()
    for task in tasks:
        print(f"{task.id}: {task.title}")
    
    # Search tasks
    results = db.search_tasks("groceries")
    
    # Delete task
    db.delete_task(task_id)
    
    # Close connection
    db.close()
```

### File Management

**Rust Example**:

```rust
use gtk::gio;
use gtk::glib;
use std::path::PathBuf;

struct FileManager;

impl FileManager {
    // Get standard directories
    fn get_documents_dir() -> PathBuf {
        glib::user_special_dir(glib::UserDirectory::Documents)
            .unwrap_or_else(|| glib::home_dir().join("Documents"))
    }
    
    fn get_pictures_dir() -> PathBuf {
        glib::user_special_dir(glib::UserDirectory::Pictures)
            .unwrap_or_else(|| glib::home_dir().join("Pictures"))
    }
    
    fn get_downloads_dir() -> PathBuf {
        glib::user_special_dir(glib::UserDirectory::Downloads)
            .unwrap_or_else(|| glib::home_dir().join("Downloads"))
    }
    
    // Application directories
    fn get_app_data_dir(app_id: &str) -> PathBuf {
        let mut path = glib::user_data_dir();
        path.push(app_id);
        std::fs::create_dir_all(&path).ok();
        path
    }
    
    fn get_app_config_dir(app_id: &str) -> PathBuf {
        let mut path = glib::user_config_dir();
        path.push(app_id);
        std::fs::create_dir_all(&path).ok();
        path
    }
    
    fn get_app_cache_dir(app_id: &str) -> PathBuf {
        let mut path = glib::user_cache_dir();
        path.push(app_id);
        std::fs::create_dir_all(&path).ok();
        path
    }
    
    // File operations
    fn read_file(path: &PathBuf) -> Result<String, String> {
        std::fs::read_to_string(path)
            .map_err(|e| format!("Failed to read file: {}", e))
    }
    
    fn write_file(path: &PathBuf, content: &str) -> Result<(), String> {
        // Ensure parent directory exists
        if let Some(parent) = path.parent() {
            std::fs::create_dir_all(parent)
                .map_err(|e| format!("Failed to create directory: {}", e))?;
        }
        
        std::fs::write(path, content)
            .map_err(|e| format!("Failed to write file: {}", e))
    }
    
    fn copy_file(from: &PathBuf, to: &PathBuf) -> Result<(), String> {
        std::fs::copy(from, to)
            .map(|_| ())
            .map_err(|e| format!("Failed to copy file: {}", e))
    }
    
    fn delete_file(path: &PathBuf) -> Result<(), String> {
        std::fs::remove_file(path)
            .map_err(|e| format!("Failed to delete file: {}", e))
    }
    
    fn file_exists(path: &PathBuf) -> bool {
        path.exists()
    }
    
    // Directory operations
    fn list_directory(path: &PathBuf) -> Result<Vec<PathBuf>, String> {
        let entries = std::fs::read_dir(path)
            .map_err(|e| format!("Failed to read directory: {}", e))?;
        
        let mut paths = Vec::new();
        for entry in entries {
            if let Ok(entry) = entry {
                paths.push(entry.path());
            }
        }
        
        Ok(paths)
    }
    
    fn create_directory(path: &PathBuf) -> Result<(), String> {
        std::fs::create_dir_all(path)
            .map_err(|e| format!("Failed to create directory: {}", e))
    }
    
    // Async file operations
    async fn read_file_async(path: PathBuf) -> Result<String, String> {
        gio::spawn_blocking(move || {
            std::fs::read_to_string(&path)
                .map_err(|e| format!("Failed to read file: {}", e))
        })
        .await
    }
    
    async fn write_file_async(path: PathBuf, content: String) -> Result<(), String> {
        gio::spawn_blocking(move || {
            if let Some(parent) = path.parent() {
                std::fs::create_dir_all(parent).ok();
            }
            std::fs::write(&path, content)
                .map_err(|e| format!("Failed to write file: {}", e))
        })
        .await
    }
}

// Usage
fn file_manager_example() {
    let app_id = "com.example.MyApp";
    
    // Get directories
    let data_dir = FileManager::get_app_data_dir(app_id);
    let config_dir = FileManager::get_app_config_dir(app_id);
    
    // Write file
    let file_path = data_dir.join("data.txt");
    FileManager::write_file(&file_path, "Hello, world!").ok();
    
    // Read file
    if let Ok(content) = FileManager::read_file(&file_path) {
        println!("Content: {}", content);
    }
    
    // List directory
    if let Ok(entries) = FileManager::list_directory(&data_dir) {
        for entry in entries {
            println!("File: {:?}", entry);
        }
    }
}
```

**Python Example**:

```python
import os
from pathlib import Path
from typing import List, Optional

class FileManager:
    """File management utilities."""
    
    @staticmethod
    def get_documents_dir() -> Path:
        """Get documents directory."""
        return Path(GLib.get_user_special_dir(GLib.UserDirectory.DIRECTORY_DOCUMENTS) 
                   or Path.home() / "Documents")
    
    @staticmethod
    def get_pictures_dir() -> Path:
        """Get pictures directory."""
        return Path(GLib.get_user_special_dir(GLib.UserDirectory.DIRECTORY_PICTURES)
                   or Path.home() / "Pictures")
    
    @staticmethod
    def get_downloads_dir() -> Path:
        """Get downloads directory."""
        return Path(GLib.get_user_special_dir(GLib.UserDirectory.DIRECTORY_DOWNLOAD)
                   or Path.home() / "Downloads")
    
    @staticmethod
    def get_app_data_dir(app_id: str) -> Path:
        """Get application data directory."""
        path = Path(GLib.get_user_data_dir()) / app_id
        path.mkdir(parents=True, exist_ok=True)
        return path
    
    @staticmethod
    def get_app_config_dir(app_id: str) -> Path:
        """Get application config directory."""
        path = Path(GLib.get_user_config_dir()) / app_id
        path.mkdir(parents=True, exist_ok=True)
        return path
    
    @staticmethod
    def get_app_cache_dir(app_id: str) -> Path:
        """Get application cache directory."""
        path = Path(GLib.get_user_cache_dir()) / app_id
        path.mkdir(parents=True, exist_ok=True)
        return path
    
    @staticmethod
    def read_file(path: Path) -> str:
        """Read file content."""
        with open(path, 'r') as f:
            return f.read()
    
    @staticmethod
    def write_file(path: Path, content: str):
        """Write content to file."""
        # Ensure parent directory exists
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, 'w') as f:
            f.write(content)
    
    @staticmethod
    def copy_file(from_path: Path, to_path: Path):
        """Copy file."""
        import shutil
        shutil.copy2(from_path, to_path)
    
    @staticmethod
    def delete_file(path: Path):
        """Delete file."""
        path.unlink()
    
    @staticmethod
    def file_exists(path: Path) -> bool:
        """Check if file exists."""
        return path.exists()
    
    @staticmethod
    def list_directory(path: Path) -> List[Path]:
        """List directory contents."""
        return list(path.iterdir())
    
    @staticmethod
    def create_directory(path: Path):
        """Create directory."""
        path.mkdir(parents=True, exist_ok=True)
    
    @staticmethod
    def read_file_async(path: Path, callback):
        """Read file asynchronously."""
        def task():
            try:
                with open(path, 'r') as f:
                    return f.read()
            except Exception as e:
                return str(e)
        
        def worker():
            result = task()
            GLib.idle_add(callback, result)
        
        import threading
        thread = threading.Thread(target=worker)
        thread.daemon = True
        thread.start()
    
    @staticmethod
    def write_file_async(path: Path, content: str, callback):
        """Write file asynchronously."""
        def task():
            try:
                path.parent.mkdir(parents=True, exist_ok=True)
                with open(path, 'w') as f:
                    f.write(content)
                return True
            except Exception as e:
                print(f"Error: {e}")
                return False
        
        def worker():
            result = task()
            GLib.idle_add(callback, result)
        
        import threading
        thread = threading.Thread(target=worker)
        thread.daemon = True
        thread.start()

# Usage
def file_manager_example():
    """File manager usage example."""
    app_id = "com.example.MyApp"
    
    # Get directories
    data_dir = FileManager.get_app_data_dir(app_id)
    config_dir = FileManager.get_app_config_dir(app_id)
    
    # Write file
    file_path = data_dir / "data.txt"
    FileManager.write_file(file_path, "Hello, world!")
    
    # Read file
    content = FileManager.read_file(file_path)
    print(f"Content: {content}")
    
    # List directory
    entries = FileManager.list_directory(data_dir)
    for entry in entries:
        print(f"File: {entry}")
```



## Chapter 12: Advanced Features

### Custom Widgets

**Rust Example**:

```rust
use gtk::glib;
use gtk::prelude::*;
use gtk::subclass::prelude::*;

// Custom widget implementation
mod imp {
    use super::*;
    use std::cell::RefCell;

    #[derive(Default)]
    pub struct ColorButton {
        pub color: RefCell<gtk::gdk::RGBA>,
    }

    #[glib::object_subclass]
    impl ObjectSubclass for ColorButton {
        const NAME: &'static str = "MyColorButton";
        type Type = super::ColorButton;
        type ParentType = gtk::Button;
    }

    impl ObjectImpl for ColorButton {
        fn constructed(&self) {
            self.parent_constructed();
            
            let obj = self.obj();
            obj.set_size_request(40, 40);
            obj.add_css_class("color-button");
            
            // Update button appearance
            self.update_color_display();
        }
        
        fn properties() -> &'static [glib::ParamSpec] {
            use once_cell::sync::Lazy;
            static PROPERTIES: Lazy<Vec<glib::ParamSpec>> = Lazy::new(|| {
                vec![
                    glib::ParamSpecBoxed::builder::<gtk::gdk::RGBA>("color")
                        .build(),
                ]
            });
            PROPERTIES.as_ref()
        }
        
        fn property(&self, _id: usize, pspec: &glib::ParamSpec) -> glib::Value {
            match pspec.name() {
                "color" => self.color.borrow().to_value(),
                _ => unimplemented!(),
            }
        }
        
        fn set_property(&self, _id: usize, value: &glib::Value, pspec: &glib::ParamSpec) {
            match pspec.name() {
                "color" => {
                    let color = value.get().unwrap();
                    self.color.replace(color);
                    self.update_color_display();
                }
                _ => unimplemented!(),
            }
        }
    }

    impl WidgetImpl for ColorButton {}
    impl ButtonImpl for ColorButton {}

    impl ColorButton {
        fn update_color_display(&self) {
            let obj = self.obj();
            let color = self.color.borrow();
            
            let css = format!(
                "button.color-button {{ background-color: rgba({},{},{},{}); }}",
                (color.red() * 255.0) as u8,
                (color.green() * 255.0) as u8,
                (color.blue() * 255.0) as u8,
                color.alpha()
            );
            
            let provider = gtk::CssProvider::new();
            provider.load_from_string(&css);
            
            obj.style_context().add_provider(
                &provider,
                gtk::STYLE_PROVIDER_PRIORITY_APPLICATION,
            );
        }
    }
}

glib::wrapper! {
    pub struct ColorButton(ObjectSubclass<imp::ColorButton>)
        @extends gtk::Button, gtk::Widget,
        @implements gtk::Accessible, gtk::Actionable, gtk::Buildable;
}

impl ColorButton {
    pub fn new() -> Self {
        glib::Object::new()
    }
    
    pub fn with_color(color: &gtk::gdk::RGBA) -> Self {
        glib::Object::builder()
            .property("color", color)
            .build()
    }
    
    pub fn color(&self) -> gtk::gdk::RGBA {
        self.property("color")
    }
    
    pub fn set_color(&self, color: &gtk::gdk::RGBA) {
        self.set_property("color", color);
    }
}

impl Default for ColorButton {
    fn default() -> Self {
        Self::new()
    }
}

// Usage
fn custom_widget_example() {
    let color_button = ColorButton::new();
    
    let red = gtk::gdk::RGBA::new(1.0, 0.0, 0.0, 1.0);
    color_button.set_color(&red);
    
    color_button.connect_clicked(|button| {
        let dialog = gtk::ColorDialog::new();
        dialog.choose_rgba(
            Some(&button.root().unwrap().downcast::<gtk::Window>().unwrap()),
            Some(&button.color()),
            None::<&gio::Cancellable>,
            glib::clone!(
                #[weak]
                button,
                move |result| {
                    if let Ok(color) = result {
                        button.set_color(&color);
                    }
                }
            ),
        );
    });
}
```

**Python Example**:

```python
from gi.repository import Gtk, Gdk, GObject

class ColorButton(Gtk.Button):
    """Custom color button widget."""
    
    __gtype_name__ = 'ColorButton'
    
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self._color = Gdk.RGBA()
        self._color.parse("red")
        
        self.set_size_request(40, 40)
        self.add_css_class("color-button")
        
        self.connect('clicked', self.on_clicked)
        self.update_color_display()
    
    @GObject.Property(type=Gdk.RGBA)
    def color(self):
        """Get current color."""
        return self._color
    
    @color.setter
    def color(self, value):
        """Set color."""
        self._color = value
        self.update_color_display()
    
    def update_color_display(self):
        """Update button appearance with current color."""
        css = f"""
        button.color-button {{
            background-color: rgba({int(self._color.red * 255)},
                                 {int(self._color.green * 255)},
                                 {int(self._color.blue * 255)},
                                 {self._color.alpha});
        }}
        """
        
        provider = Gtk.CssProvider()
        provider.load_from_string(css)
        
        self.get_style_context().add_provider(
            provider,
            Gtk.STYLE_PROVIDER_PRIORITY_APPLICATION
        )
    
    def on_clicked(self, button):
        """Handle button click."""
        dialog = Gtk.ColorDialog()
        dialog.choose_rgba(
            self.get_root(),
            self._color,
            None,
            self.on_color_chosen
        )
    
    def on_color_chosen(self, dialog, result):
        """Handle color chosen."""
        try:
            color = dialog.choose_rgba_finish(result)
            self.color = color
        except Exception as e:
            print(f"Color selection cancelled: {e}")

# Usage
def custom_widget_example():
    """Custom widget usage example."""
    color_button = ColorButton()
    
    red = Gdk.RGBA()
    red.parse("red")
    color_button.color = red
```

### Drag and Drop

**Rust Example**:

```rust
use gtk::gdk;
use gtk::gio;
use gtk::glib;
use gtk::prelude::*;

fn setup_drag_source(widget: &impl IsA<gtk::Widget>) {
    let drag_source = gtk::DragSource::new();
    drag_source.set_actions(gdk::DragAction::COPY);
    
    // Prepare drag data
    drag_source.connect_prepare(|source, _x, _y| {
        let value = glib::Value::from("Dragged text");
        Some(gdk::ContentProvider::for_value(&value))
    });
    
    // Drag begin
    drag_source.connect_drag_begin(|source, drag| {
        // Set drag icon
        let paintable = gtk::WidgetPaintable::new(Some(source.widget()));
        source.set_icon(Some(&paintable), 0, 0);
    });
    
    // Drag end
    drag_source.connect_drag_end(|_source, _drag, _delete_data| {
        println!("Drag ended");
    });
    
    widget.add_controller(drag_source);
}

fn setup_drop_target(widget: &impl IsA<gtk::Widget>) {
    let drop_target = gtk::DropTarget::new(glib::Type::STRING, gdk::DragAction::COPY);
    
    // Drop event
    drop_target.connect_drop(|_target, value, _x, _y| {
        if let Ok(text) = value.get::<String>() {
            println!("Dropped: {}", text);
            return true;
        }
        false
    });
    
    // Drag enter
    drop_target.connect_enter(|target, _x, _y| {
        target.widget().add_css_class("drag-hover");
        gdk::DragAction::COPY
    });
    
    // Drag leave
    drop_target.connect_leave(|target| {
        target.widget().remove_css_class("drag-hover");
    });
    
    widget.add_controller(drop_target);
}

// File drag and drop
fn setup_file_drop_target(widget: &impl IsA<gtk::Widget>) {
    let drop_target = gtk::DropTarget::new(gio::File::static_type(), gdk::DragAction::COPY);
    
    drop_target.connect_drop(|_target, value, _x, _y| {
        if let Ok(file) = value.get::<gio::File>() {
            if let Some(path) = file.path() {
                println!("Dropped file: {:?}", path);
                return true;
            }
        }
        false
    });
    
    widget.add_controller(drop_target);
}
```

**Python Example**:

```python
def setup_drag_source(self, widget):
    """Setup drag source for widget."""
    drag_source = Gtk.DragSource()
    drag_source.set_actions(Gdk.DragAction.COPY)
    
    # Prepare drag data
    def on_prepare(source, x, y):
        value = GObject.Value(str, "Dragged text")
        return Gdk.ContentProvider.new_for_value(value)
    
    drag_source.connect('prepare', on_prepare)
    
    # Drag begin
    def on_drag_begin(source, drag):
        paintable = Gtk.WidgetPaintable.new(source.get_widget())
        source.set_icon(paintable, 0, 0)
    
    drag_source.connect('drag-begin', on_drag_begin)
    
    # Drag end
    def on_drag_end(source, drag, delete_data):
        print("Drag ended")
    
    drag_source.connect('drag-end', on_drag_end)
    
    widget.add_controller(drag_source)

def setup_drop_target(self, widget):
    """Setup drop target for widget."""
    drop_target = Gtk.DropTarget.new(str, Gdk.DragAction.COPY)
    
    # Drop event
    def on_drop(target, value, x, y):
        if isinstance(value, str):
            print(f"Dropped: {value}")
            return True
        return False
    
    drop_target.connect('drop', on_drop)
    
    # Drag enter
    def on_enter(target, x, y):
        target.get_widget().add_css_class('drag-hover')
        return Gdk.DragAction.COPY
    
    drop_target.connect('enter', on_enter)
    
    # Drag leave
    def on_leave(target):
        target.get_widget().remove_css_class('drag-hover')
    
    drop_target.connect('leave', on_leave)
    
    widget.add_controller(drop_target)

def setup_file_drop_target(self, widget):
    """Setup file drop target."""
    drop_target = Gtk.DropTarget.new(Gio.File, Gdk.DragAction.COPY)
    
    def on_drop(target, value, x, y):
        if isinstance(value, Gio.File):
            path = value.get_path()
            print(f"Dropped file: {path}")
            return True
        return False
    
    drop_target.connect('drop', on_drop)
    widget.add_controller(drop_target)
```

### Animations

**Rust Example**:

```rust
use adwaita::prelude::*;
use gtk::glib;

// Simple fade animation
fn fade_in_widget(widget: &impl IsA<gtk::Widget>) {
    widget.set_opacity(0.0);
    
    let target = adwaita::PropertyAnimationTarget::new(widget, "opacity");
    let animation = adwaita::TimedAnimation::new(
        widget,
        0.0,  // from
        1.0,  // to
        500,  // duration in ms
        target,
    );
    
    animation.play();
}

fn fade_out_widget(widget: &impl IsA<gtk::Widget>) {
    let target = adwaita::PropertyAnimationTarget::new(widget, "opacity");
    let animation = adwaita::TimedAnimation::new(
        widget,
        widget.opacity(),
        0.0,
        500,
        target,
    );
    
    animation.play();
}

// Spring animation (bouncy effect)
fn spring_animation(widget: &impl IsA<gtk::Widget>) {
    let target = adwaita::PropertyAnimationTarget::new(widget, "scale-x");
    
    let animation = adwaita::SpringAnimation::new(
        widget,
        widget.scale_x(),  // from
        2.0,               // to
        adwaita::SpringParams::new(1.0, 0.5, 500.0),
        target,
    );
    
    animation.play();
}

// Rotation animation
fn rotate_widget(widget: &impl IsA<gtk::Widget>, degrees: f64) {
    // Note: Need to use CSS transform for rotation
    // This example shows how to animate a custom property
    
    let target = adwaita::CallbackAnimationTarget::new(
        glib::clone!(
            #[weak]
            widget,
            move |value| {
                let angle = value * degrees;
                // Apply rotation transform
                let transform = gtk::gsk::Transform::new().rotate(angle as f32);
                // This would need proper implementation with paintable
            }
        )
    );
    
    let animation = adwaita::TimedAnimation::new(
        widget,
        0.0,
        1.0,
        1000,
        target,
    );
    
    animation.play();
}

// Slide animation
fn slide_in_from_bottom(widget: &impl IsA<gtk::Widget>) {
    let height = widget.allocated_height() as f64;
    widget.set_margin_bottom(-(height as i32));
    
    let target = adwaita::PropertyAnimationTarget::new(widget, "margin-bottom");
    let animation = adwaita::TimedAnimation::new(
        widget,
        -height,
        0.0,
        300,
        target,
    );
    
    animation.set_easing(adwaita::Easing::EaseOutCubic);
    animation.play();
}

// Chain animations
fn chain_animations(widget: &impl IsA<gtk::Widget>) {
    let target1 = adwaita::PropertyAnimationTarget::new(widget, "opacity");
    let anim1 = adwaita::TimedAnimation::new(widget, 1.0, 0.0, 200, target1);
    
    anim1.connect_done(glib::clone!(
        #[weak]
        widget,
        move |_| {
            // Start second animation when first is done
            let target2 = adwaita::PropertyAnimationTarget::new(&widget, "opacity");
            let anim2 = adwaita::TimedAnimation::new(&widget, 0.0, 1.0, 200, target2);
            anim2.play();
        }
    ));
    
    anim1.play();
}

// Animation with custom easing
fn custom_easing_animation(widget: &impl IsA<gtk::Widget>) {
    let target = adwaita::PropertyAnimationTarget::new(widget, "opacity");
    let animation = adwaita::TimedAnimation::new(widget, 0.0, 1.0, 800, target);
    
    // Available easing functions:
    // Linear, EaseInQuad, EaseOutQuad, EaseInOutQuad
    // EaseInCubic, EaseOutCubic, EaseInOutCubic
    // EaseInQuart, EaseOutQuart, EaseInOutQuart
    // EaseInQuint, EaseOutQuint, EaseInOutQuint
    // EaseInSine, EaseOutSine, EaseInOutSine
    // EaseInExpo, EaseOutExpo, EaseInOutExpo
    // EaseInCirc, EaseOutCirc, EaseInOutCirc
    // EaseInElastic, EaseOutElastic, EaseInOutElastic
    // EaseInBack, EaseOutBack, EaseInOutBack
    // EaseInBounce, EaseOutBounce, EaseInOutBounce
    
    animation.set_easing(adwaita::Easing::EaseOutBounce);
    animation.play();
}

// Pause and resume animations
fn animation_controls(animation: &adwaita::Animation) {
    // Pause
    animation.pause();
    
    // Resume
    animation.resume();
    
    // Reset
    animation.reset();
    
    // Skip to end
    animation.skip();
}
```

**Python Example**:

```python
def fade_in_widget(self, widget):
    """Fade in a widget."""
    widget.set_opacity(0.0)
    
    target = Adw.PropertyAnimationTarget.new(widget, "opacity")
    animation = Adw.TimedAnimation.new(
        widget,
        0.0,  # from
        1.0,  # to
        500,  # duration in ms
        target
    )
    
    animation.play()

def fade_out_widget(self, widget):
    """Fade out a widget."""
    target = Adw.PropertyAnimationTarget.new(widget, "opacity")
    animation = Adw.TimedAnimation.new(
        widget,
        widget.get_opacity(),
        0.0,
        500,
        target
    )
    
    animation.play()

def spring_animation(self, widget):
    """Spring animation with bouncy effect."""
    target = Adw.PropertyAnimationTarget.new(widget, "scale-x")
    
    spring_params = Adw.SpringParams.new(1.0, 0.5, 500.0)
    animation = Adw.SpringAnimation.new(
        widget,
        widget.get_scale_x(),
        2.0,
        spring_params,
        target
    )
    
    animation.play()

def slide_in_from_bottom(self, widget):
    """Slide widget in from bottom."""
    height = widget.get_allocated_height()
    widget.set_margin_bottom(-height)
    
    target = Adw.PropertyAnimationTarget.new(widget, "margin-bottom")
    animation = Adw.TimedAnimation.new(
        widget,
        -height,
        0.0,
        300,
        target
    )
    
    animation.set_easing(Adw.Easing.EASE_OUT_CUBIC)
    animation.play()

def chain_animations(self, widget):
    """Chain multiple animations."""
    target1 = Adw.PropertyAnimationTarget.new(widget, "opacity")
    anim1 = Adw.TimedAnimation.new(widget, 1.0, 0.0, 200, target1)
    
    def on_done(animation):
        # Start second animation when first is done
        target2 = Adw.PropertyAnimationTarget.new(widget, "opacity")
        anim2 = Adw.TimedAnimation.new(widget, 0.0, 1.0, 200, target2)
        anim2.play()
    
    anim1.connect('done', on_done)
    anim1.play()

def custom_easing_animation(self, widget):
    """Animation with custom easing."""
    target = Adw.PropertyAnimationTarget.new(widget, "opacity")
    animation = Adw.TimedAnimation.new(widget, 0.0, 1.0, 800, target)
    
    # Available easing functions:
    # LINEAR, EASE_IN_QUAD, EASE_OUT_QUAD, EASE_IN_OUT_QUAD
    # EASE_IN_CUBIC, EASE_OUT_CUBIC, EASE_IN_OUT_CUBIC
    # EASE_IN_QUART, EASE_OUT_QUART, EASE_IN_OUT_QUART
    # EASE_IN_QUINT, EASE_OUT_QUINT, EASE_IN_OUT_QUINT
    # EASE_IN_SINE, EASE_OUT_SINE, EASE_IN_OUT_SINE
    # EASE_IN_EXPO, EASE_OUT_EXPO, EASE_IN_OUT_EXPO
    # EASE_IN_CIRC, EASE_OUT_CIRC, EASE_IN_OUT_CIRC
    # EASE_IN_ELASTIC, EASE_OUT_ELASTIC, EASE_IN_OUT_ELASTIC
    # EASE_IN_BACK, EASE_OUT_BACK, EASE_IN_OUT_BACK
    # EASE_IN_BOUNCE, EASE_OUT_BOUNCE, EASE_IN_OUT_BOUNCE
    
    animation.set_easing(Adw.Easing.EASE_OUT_BOUNCE)
    animation.play()

def animation_controls(self, animation):
    """Control animation playback."""
    # Pause
    animation.pause()
    
    # Resume
    animation.resume()
    
    # Reset
    animation.reset()
    
    # Skip to end
    animation.skip()
```

### Custom CSS Styling

**Rust Example**:

```rust
use gtk::prelude::*;

fn load_custom_css() {
    let provider = gtk::CssProvider::new();
    
    let css = r#"
        /* Custom button styling */
        .custom-button {
            background: linear-gradient(to bottom, #4a90e2, #357abd);
            color: white;
            border-radius: 8px;
            padding: 12px 24px;
            font-weight: bold;
        }
        
        .custom-button:hover {
            background: linear-gradient(to bottom, #5ba3f5, #4a90e2);
        }
        
        .custom-button:active {
            background: linear-gradient(to bottom, #357abd, #2868a8);
        }
        
        /* Custom card styling */
        .card {
            background-color: @card_bg_color;
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        }
        
        /* Dark mode support */
        .card:backdrop {
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
        }
        
        /* Custom list styling */
        .custom-list row {
            border-radius: 8px;
            margin: 4px 0;
        }
        
        .custom-list row:hover {
            background-color: alpha(@accent_color, 0.1);
        }
        
        /* Animated progress */
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        
        .pulsing {
            animation: pulse 2s ease-in-out infinite;
        }
        
        /* Custom separator */
        .fancy-separator {
            background: linear-gradient(to right, 
                transparent, 
                @accent_color, 
                transparent);
            min-height: 2px;
        }
    "#;
    
    provider.load_from_string(css);
    
    gtk::style_context_add_provider_for_display(
        &gtk::gdk::Display::default().unwrap(),
        &provider,
        gtk::STYLE_PROVIDER_PRIORITY_APPLICATION,
    );
}

fn apply_custom_style_to_widget(widget: &impl IsA<gtk::Widget>) {
    widget.add_css_class("custom-button");
}

fn load_css_from_resource(resource_path: &str) {
    let provider = gtk::CssProvider::new();
    provider.load_from_resource(resource_path);
    
    gtk::style_context_add_provider_for_display(
        &gtk::gdk::Display::default().unwrap(),
        &provider,
        gtk::STYLE_PROVIDER_PRIORITY_APPLICATION,
    );
}
```

**Python Example**:

```python
def load_custom_css(self):
    """Load custom CSS styling."""
    provider = Gtk.CssProvider()
    
    css = """
        /* Custom button styling */
        .custom-button {
            background: linear-gradient(to bottom, #4a90e2, #357abd);
            color: white;
            border-radius: 8px;
            padding: 12px 24px;
            font-weight: bold;
        }
        
        .custom-button:hover {
            background: linear-gradient(to bottom, #5ba3f5, #4a90e2);
        }
        
        .custom-button:active {
            background: linear-gradient(to bottom, #357abd, #2868a8);
        }
        
        /* Custom card styling */
        .card {
            background-color: @card_bg_color;
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        }
        
        /* Dark mode support */
        .card:backdrop {
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
        }
        
        /* Custom list styling */
        .custom-list row {
            border-radius: 8px;
            margin: 4px 0;
        }
        
        .custom-list row:hover {
            background-color: alpha(@accent_color, 0.1);
        }
        
        /* Animated progress */
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        
        .pulsing {
            animation: pulse 2s ease-in-out infinite;
        }
        
        /* Custom separator */
        .fancy-separator {
            background: linear-gradient(to right, 
                transparent, 
                @accent_color, 
                transparent);
            min-height: 2px;
        }
    """
    
    provider.load_from_string(css)
    
    Gtk.StyleContext.add_provider_for_display(
        Gdk.Display.get_default(),
        provider,
        Gtk.STYLE_PROVIDER_PRIORITY_APPLICATION
    )

def apply_custom_style_to_widget(self, widget):
    """Apply custom style to widget."""
    widget.add_css_class("custom-button")

def load_css_from_resource(self, resource_path):
    """Load CSS from resource."""
    provider = Gtk.CssProvider()
    provider.load_from_resource(resource_path)
    
    Gtk.StyleContext.add_provider_for_display(
        Gdk.Display.get_default(),
        provider,
        Gtk.STYLE_PROVIDER_PRIORITY_APPLICATION
    )
```

### Accessibility

**Rust Example**:

```rust
use gtk::prelude::*;

fn setup_accessibility(widget: &impl IsA<gtk::Widget>) {
    // Set accessible label
    widget.set_accessible_label(Some("User Profile Picture"));
    
    // Set accessible description
    widget.set_accessible_description(Some("Click to change your profile picture"));
    
    // Set accessible role
    widget.set_accessible_role(gtk::AccessibleRole::Button);
}

fn setup_custom_button_accessibility(button: &gtk::Button) {
    button.set_accessible_label(Some("Delete Item"));
    button.set_tooltip_text(Some("Delete this item permanently"));
    
    // For icon-only buttons, always set accessible label
    button.set_icon_name("user-trash-symbolic");
}

fn setup_form_accessibility(entry: &gtk::Entry, label: &gtk::Label) {
    // Associate label with entry
    label.set_mnemonic_widget(Some(entry));
    label.set_use_underline(true);
    label.set_text("_Username:");
    
    // Set placeholder for additional context
    entry.set_placeholder_text(Some("Enter your username"));
}

fn setup_image_accessibility(picture: &gtk::Picture) {
    // Always provide alt text for images
    picture.set_accessible_label(Some("GNOME logo: a blue and gray footprint"));
    picture.set_accessible_role(gtk::AccessibleRole::Img);
}

fn announce_to_screen_reader(widget: &impl IsA<gtk::Widget>, message: &str) {
    // Create an invisible label for announcements
    let announcement = gtk::Label::new(Some(message));
    announcement.set_accessible_role(gtk::AccessibleRole::Alert);
    
    // This will be announced by screen readers
    // Add to widget hierarchy temporarily
}
```

**Python Example**:

```python
def setup_accessibility(self, widget):
    """Setup accessibility for widget."""
    # Set accessible label
    widget.set_accessible_label("User Profile Picture")
    
    # Set accessible description
    widget.set_accessible_description("Click to change your profile picture")
    
    # Set accessible role
    widget.set_accessible_role(Gtk.AccessibleRole.BUTTON)

def setup_custom_button_accessibility(self, button):
    """Setup accessibility for custom button."""
    button.set_accessible_label("Delete Item")
    button.set_tooltip_text("Delete this item permanently")
    
    # For icon-only buttons, always set accessible label
    button.set_icon_name("user-trash-symbolic")

def setup_form_accessibility(self, entry, label):
    """Setup form field accessibility."""
    # Associate label with entry
    label.set_mnemonic_widget(entry)
    label.set_use_underline(True)
    label.set_text("_Username:")
    
    # Set placeholder for additional context
    entry.set_placeholder_text("Enter your username")

def setup_image_accessibility(self, picture):
    """Setup image accessibility."""
    # Always provide alt text for images
    picture.set_accessible_label("GNOME logo: a blue and gray footprint")
    picture.set_accessible_role(Gtk.AccessibleRole.IMG)

def announce_to_screen_reader(self, widget, message):
    """Announce message to screen reader."""
    # Create an invisible label for announcements
    announcement = Gtk.Label(label=message)
    announcement.set_accessible_role(Gtk.AccessibleRole.ALERT)
    
    # This will be announced by screen readers
```



## Chapter 13: Packaging and Distribution

### Desktop File

**com.example.MyApp.desktop.in**:

```desktop
[Desktop Entry]
Name=My Application
GenericName=Task Manager
Comment=Manage your tasks efficiently
# Translators: These are keywords used to search for the application
Keywords=tasks;todo;organizer;productivity;
Exec=my-app
Icon=com.example.MyApp
Terminal=false
Type=Application
Categories=GTK;GNOME;Utility;
StartupNotify=true
X-GNOME-UsesNotifications=true
# For handling custom URIs
MimeType=x-scheme-handler/myapp;
```

### MetaInfo File

**com.example.MyApp.metainfo.xml.in**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component type="desktop-application">
  <id>com.example.MyApp</id>
  
  <name>My Application</name>
  <summary>A simple task management application</summary>
  
  <metadata_license>CC0-1.0</metadata_license>
  <project_license>GPL-3.0-or-later</project_license>
  
  <description>
    <p>
      My Application is a modern, easy-to-use task manager for GNOME.
      It helps you organize your daily tasks and stay productive.
    </p>
    <p>Features:</p>
    <ul>
      <li>Create and organize tasks</li>
      <li>Set reminders and due dates</li>
      <li>Mark tasks as complete</li>
      <li>Search and filter tasks</li>
      <li>Dark mode support</li>
    </ul>
  </description>
  
  <launchable type="desktop-id">com.example.MyApp.desktop</launchable>
  
  <screenshots>
    <screenshot type="default">
      <image>https://example.com/screenshots/main.png</image>
      <caption>Main window</caption>
    </screenshot>
    <screenshot>
      <image>https://example.com/screenshots/task-detail.png</image>
      <caption>Task details</caption>
    </screenshot>
  </screenshots>
  
  <url type="homepage">https://example.com/my-app</url>
  <url type="bugtracker">https://github.com/example/my-app/issues</url>
  <url type="donation">https://example.com/donate</url>
  <url type="help">https://example.com/my-app/help</url>
  <url type="translate">https://example.com/my-app/translate</url>
  
  <developer_name>Your Name</developer_name>
  <developer id="com.example">
    <name>Your Name</name>
  </developer>
  
  <update_contact>your.email@example.com</update_contact>
  
  <releases>
    <release version="1.0.0" date="2025-01-15">
      <description>
        <p>Initial release</p>
        <ul>
          <li>Create and manage tasks</li>
          <li>Set reminders</li>
          <li>Dark mode support</li>
        </ul>
      </description>
    </release>
  </releases>
  
  <content_rating type="oars-1.1"/>
  
  <recommends>
    <control>keyboard</control>
    <control>pointing</control>
    <control>touch</control>
  </recommends>
  
  <requires>
    <display_length compare="ge">360</display_length>
  </requires>
  
  <branding>
    <color type="primary" scheme_preference="light">#3584e4</color>
    <color type="primary" scheme_preference="dark">#1c71d8</color>
  </branding>
</component>
```

### Flatpak Manifest

**com.example.MyApp.json**:

```json
{
    "app-id": "com.example.MyApp",
    "runtime": "org.gnome.Platform",
    "runtime-version": "47",
    "sdk": "org.gnome.Sdk",
    "sdk-extensions": [
        "org.freedesktop.Sdk.Extension.rust-stable"
    ],
    "command": "my-app",
    "finish-args": [
        "--share=ipc",
        "--socket=fallback-x11",
        "--socket=wayland",
        "--device=dri",
        "--share=network",
        "--filesystem=xdg-documents",
        "--filesystem=xdg-download",
        "--talk-name=org.freedesktop.Notifications",
        "--talk-name=org.freedesktop.secrets"
    ],
    "build-options": {
        "append-path": "/usr/lib/sdk/rust-stable/bin",
        "env": {
            "CARGO_HOME": "/run/build/my-app/cargo"
        }
    },
    "modules": [
        {
            "name": "my-app",
            "buildsystem": "meson",
            "config-opts": [
                "-Dprofile=release"
            ],
            "sources": [
                {
                    "type": "git",
                    "url": "https://github.com/example/my-app.git",
                    "tag": "v1.0.0"
                }
            ]
        }
    ]
}
```

### Building on Arch Linux

```bash
# Install build dependencies
sudo pacman -S meson ninja gtk4 libadwaita rust cargo

# Clone repository
git clone https://github.com/example/my-app.git
cd my-app

# Build with Meson
meson setup builddir --prefix=/usr
meson compile -C builddir

# Install
sudo meson install -C builddir

# Or create a PKGBUILD for Arch
```

### PKGBUILD for Arch Linux

**PKGBUILD**:

```bash
# Maintainer: Your Name <your.email@example.com>

pkgname=my-app
pkgver=1.0.0
pkgrel=1
pkgdesc="A simple task management application for GNOME"
arch=('x86_64')
url="https://github.com/example/my-app"
license=('GPL-3.0-or-later')
depends=('gtk4' 'libadwaita' 'sqlite')
makedepends=('meson' 'cargo' 'rust')
source=("$pkgname-$pkgver.tar.gz::$url/archive/v$pkgver.tar.gz")
sha256sums=('SKIP')

build() {
    cd "$pkgname-$pkgver"
    arch-meson . build
    meson compile -C build
}

check() {
    cd "$pkgname-$pkgver"
    meson test -C build --print-errorlogs
}

package() {
    cd "$pkgname-$pkgver"
    meson install -C build --destdir "$pkgdir"
}
```

### Building Flatpak

```bash
# Install flatpak and flatpak-builder
sudo pacman -S flatpak flatpak-builder

# Add Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Install GNOME SDK and Platform
flatpak install flathub org.gnome.Platform//47 org.gnome.Sdk//47

# Build Flatpak
flatpak-builder --user --install --force-clean build-dir com.example.MyApp.json

# Run Flatpak
flatpak run com.example.MyApp

# Export as bundle
flatpak-builder --repo=repo --force-clean build-dir com.example.MyApp.json
flatpak build-bundle repo my-app.flatpak com.example.MyApp

# Test the bundle
flatpak install my-app.flatpak
```

### Continuous Integration

**.github/workflows/ci.yml**:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/gnome/gnome-runtime-images/gnome:47
      
    steps:
    - uses: actions/checkout@v3
    
    - name: Install dependencies
      run: |
        dnf install -y meson ninja-build gtk4-devel libadwaita-devel \
                       rust cargo sqlite-devel
    
    - name: Build
      run: |
        meson setup builddir
        meson compile -C builddir
    
    - name: Test
      run: |
        meson test -C builddir --print-errorlogs
    
    - name: Build Flatpak
      uses: flatpak/flatpak-github-actions/flatpak-builder@v6
      with:
        bundle: my-app.flatpak
        manifest-path: com.example.MyApp.json
        cache-key: flatpak-builder-${{ github.sha }}
```

### Translation Support

**po/POTFILES.in**:

```
src/main.rs
src/window.rs
src/application.rs
data/com.example.MyApp.desktop.in
data/com.example.MyApp.metainfo.xml.in
data/ui/window.ui
```

**po/LINGUAS**:

```
de
es
fr
pt_BR
ru
zh_CN
```

**Extract translations**:

```bash
# Generate POT file
xgettext --files-from=po/POTFILES.in --output=po/my-app.pot

# Update PO files
for lang in $(cat po/LINGUAS); do
    msgmerge --update po/$lang.po po/my-app.pot
done
```



## Conclusion

This handbook has covered the essential aspects of creating Libadwaita applications for GNOME:

1. **Foundation**: Understanding Libadwaita, GNOME HIG, and development environment setup
2. **Application Structure**: Proper project organization, naming conventions, and build systems
3. **Widgets**: From simple to complex, understanding the full widget ecosystem
4. **Layouts**: Organizing content with modern, adaptive containers
5. **Patterns**: Implementing standard GNOME application patterns
6. **System Integration**: D-Bus, notifications, portals, and system settings
7. **Data Management**: GSettings, databases, and file handling
8. **Advanced Features**: Custom widgets, animations, drag and drop, and accessibility
9. **Distribution**: Packaging for Flatpak and native Linux distributions

### Best Practices Summary

- **Follow GNOME HIG**: Your app should feel at home in GNOME
- **Use Libadwaita widgets**: They provide consistency and adaptability
- **Support dark mode**: It's automatic with Libadwaita
- **Make it accessible**: Use proper labels, roles, and keyboard navigation
- **Test on mobile**: Libadwaita apps should work on phones too
- **Respect user preferences**: Use GSettings and system settings
- **Handle errors gracefully**: Show helpful messages with toasts and dialogs
- **Provide keyboard shortcuts**: Power users will thank you
- **Document your code**: Future you will thank you
- **Package as Flatpak**: It's the preferred distribution method for GNOME apps

### Resources

- **GNOME Developer Documentation**: https://developer.gnome.org
- **Libadwaita Documentation**: https://gnome.pages.gitlab.gnome.org/libadwaita/doc/
- **GNOME Human Interface Guidelines**: https://developer.gnome.org/hig/
- **GTK Documentation**: https://docs.gtk.org/gtk4/
- **Rust GTK Bindings**: https://gtk-rs.org/
- **Python GTK Bindings**: https://pygobject.readthedocs.io/
- **GNOME Builder**: The official IDE for GNOME development
- **Flatpak Documentation**: https://docs.flatpak.org/

### Community

- **GNOME Discourse**: https://discourse.gnome.org/
- **Matrix Chat**: #gnome-hackers:gnome.org
- **GitLab**: https://gitlab.gnome.org/
- **GNOME Circle**: Submit your app to be part of the GNOME ecosystem

**Happy coding! Welcome to the GNOME developer community!**
