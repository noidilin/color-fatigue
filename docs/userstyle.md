# Catppuccin Userstyles Customization Workflow

## Overview

This workflow allows you to automatically download, customize, and maintain Catppuccin userstyles with your preferred accent color and custom `lib.less` URL. The customized styles are saved to a local file that can be imported into the Stylus browser extension.

## Prerequisites

- Nushell shell
- Stylus browser extension installed
- Internet connection to fetch the latest userstyles

## Custom Functions

Two Nushell functions are defined in your config for managing Catppuccin userstyles:

```nu
def stylus-update [
  accent: string = 'rosewater'
  url: string = 'https://github.com/catppuccin/userstyles/releases/download/all-userstyles-export/import.json'
  lib: string = 'https://noidilin.github.io/color-fatigue/lib/lib.less'
] {
  let file = ( $env.USERPROFILE | path join '.local/etc/stylus/color-fatigue.json' )
  
  # First, let's see what we're working with
  let data = (http get $url | decode utf-8 | from json)
  
  # Check if it's wrapped in an object or is a direct array
  let json_data = (
    if ($data | describe) =~ "list" {
      # It's an array, process directly
      $data | each { |style| 
        if ($style | get --optional usercssData.vars.accentColor) != null {
          $style | update usercssData.vars.accentColor.value $accent
        } else {
          $style
        }
      }
    } else {
      # It might be wrapped, adjust as needed
      $data
    }
    | to json --indent 2
  )
  
  $json_data
    | str replace --all "https://userstyles.catppuccin.com/lib/lib.less" $lib
    | save --force $file
  
  print $"✓ Created ($file) with ($accent) accent"
}

def stylus-examine [] {
  let file_content = (open ~/.local/etc/stylus/color-fatigue.json)

  print "=== Verification Summary ==="
  print $"Total styles: ($file_content | length)"
  print $"Accent colors: ($file_content | each { |s| $s.usercssData?.vars?.accentColor?.value } | uniq)"
  print $"Custom lib URL present: ($file_content | to json | str contains 'noidilin.github.io/color-fatigue')"
  print $"Old lib URL present: ($file_content | to json | str contains 'userstyles.catppuccin.com')"
}
```

### 1. `stylus-update`

Downloads and customizes the Catppuccin userstyles with your preferences.

**Parameters:**

- `accent` (optional, default: `'rosewater'`) - The accent color for all userstyles
  - Available colors: `rosewater`, `flamingo`, `pink`, `mauve`, `red`, `maroon`, `peach`, `yellow`, `green`, `teal`, `sky`, `sapphire`, `blue`, `lavender`
- `url` (optional, default: GitHub release URL) - Source URL for the `import.json` file
- `lib` (optional, default: your custom URL) - Custom `lib.less` URL to replace the original

**What it does:**

1. Downloads the latest `import.json` from Catppuccin userstyles releases
2. Parses the JSON and updates the accent color for all userstyles that support it
3. Replaces all references to the official `lib.less` with your custom hosted version
4. Saves the customized file to `~/.local/etc/stylus/color-fatigue.json`

### 2. `stylus-examine`

Verifies that the customization was applied correctly.

**What it shows:**

- Total number of styles in the file
- All unique accent colors found (should show only your chosen color)
- Whether your custom lib URL is present (should be `true`)
- Whether the old official lib URL is present (should be `false`)

## Usage Workflow

### Step 1: Update Userstyles

Run the update function with your preferred accent color:

```nu
# Use default settings (rosewater accent)
stylus-update

# Use a different accent color
stylus-update mauve
stylus-update peach
stylus-update lavender

# Override URLs if needed
stylus-update blue --url "https://example.com/import.json" --lib "https://example.com/lib.less"
```

### Step 2: Verify Changes

Check that the modifications were applied correctly:

```nu
stylus-examine
```

Expected output:

```
=== Verification Summary ===
Total styles: [number]
Accent colors: [your chosen color]
Custom lib URL present: true
Old lib URL present: false
```

### Step 3: Import to Stylus

1. Open the Stylus browser extension
2. Click the **Manage** button
3. In the sidebar, find the **Backup** section
4. Click **Import** and select the file:
   - Location: `C:\Users\[YourUsername]\.local\etc\stylus\color-fatigue.json`
   - Or use the path: `~/.local/etc/stylus/color-fatigue.json`
5. Enable **CSP Patching** in Stylus Settings > Advanced

### Step 4: Enable Userstyles

After importing, you can:

- Enable/disable individual userstyles from the Stylus manage page
- Configure additional per-site options if available
- Adjust light/dark flavor preferences

## Maintenance

### Regular Updates

To get the latest userstyles from Catppuccin:

```nu
# Fetch latest and apply your customizations
stylus-update

# Verify the update
stylus-examine

# Re-import to Stylus (overwrites previous import)
```

### Changing Accent Colors

Simply run `stylus-update` with a different accent color and re-import:

```nu
stylus-update sapphire
stylus-examine
# Then re-import to Stylus
```

### Troubleshooting

**Function not found error:**

- Ensure the functions are defined in your Nushell config
- Reload config: `source ~/.config/nushell/config.nu`

**Network errors:**

- Check internet connection
- Verify the GitHub release URL is accessible

**Import fails in Stylus:**

- Ensure the file exists at the expected location
- Check that the JSON is valid: `open ~/.local/etc/stylus/color-fatigue.json | from json`

**Styles not applying:**

- Enable CSP Patching in Stylus settings
- Check that the userstyle is enabled for the website
- Verify your custom `lib.less` URL is accessible

## Technical Details

### Why Custom lib.less?

The `lib.less` file contains the core Catppuccin color variables and mixins. By hosting your own version at `https://noidilin.github.io/color-fatigue/lib/lib.less`, you can:

- Customize the base color palette
- Modify theme behavior globally
- Maintain consistent styling across all userstyles
- Work offline or with custom CDN

### JSON Structure

The `import.json` file contains an array of userstyle objects. Each object has:

```javascript
{
  "name": "Site Name",
  "usercssData": {
    "vars": {
      "accentColor": {
        "value": "rosewater"  // This is what gets modified
      }
    }
  },
  "sections": [
    // CSS with @import "https://userstyles.catppuccin.com/lib/lib.less"
    // This URL gets replaced with your custom one
  ]
}
```

### Function Logic

The `stylus-update` function:

1. Fetches binary data from URL
2. Decodes to UTF-8 string
3. Parses as JSON
4. Checks if data is a list (array)
5. Iterates each style and updates `accentColor.value` if it exists
6. Converts back to formatted JSON string
7. Replaces all lib.less URLs using string replacement
8. Saves to local file

## Benefits

✅ **Automation**: One command updates all styles with your preferences  
✅ **Consistency**: All userstyles use the same accent color  
✅ **Customization**: Use your own hosted lib.less for global theme tweaks  
✅ **Version Control**: Keep your customized file in source control  
✅ **Repeatability**: Easy to recreate your setup on new machines  

## Future Enhancements

Potential additions to consider:

- Add flavor (light/dark theme) customization
- Filter specific sites to include/exclude
- Backup previous versions before updating
- Automatic import to Stylus via file watching
- Multiple color schemes (e.g., different accents per site category)
