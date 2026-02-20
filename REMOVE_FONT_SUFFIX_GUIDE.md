# Guide: Remove Font Name Suffixes (NF/CN)

This guide explains how to modify the Maple Font build scripts to remove automatic suffix additions to font names, ensuring the generated fonts use exactly the name specified in `config.json`.

## Purpose

By default, the build script automatically appends suffixes to font names to distinguish between different variants:
- **NF** suffix for Nerd Font versions (e.g., "Maple Mono" → "Maple Mono NF")
- **CN** suffix for Chinese versions (e.g., "Maple Mono" → "Maple Mono CN") 
- **NF CN** combined suffix for Nerd Font + Chinese versions

These modifications ensure that:
1. The font family name (NameID 1) uses only your configured `family_name`
2. The font full name (NameID 4) has no variant suffixes
3. The PostScript name (NameID 6) is clean and suffix-free
4. The preferred family name (NameID 16) matches your configuration
5. All intermediate/base fonts also follow the same naming convention

## Files to Modify

1. **`build.py`** - Main build script with NF and CN font naming functions
2. **`source/py/task/nerdfont.py`** - Nerd Font base font generator

## Step-by-Step Instructions

### Step 1: Modify Nerd Font naming in `build.py`

Find the `build_nf()` function around line 1240. Make these changes:

**Change 1:** Set `nf_sym` to empty string and simplify PostScript name

```python
# Original:
nf_sym = f"NF{font_config.get_nf_suffix()}"
postscript_name = f"{font_config.family_name_compact}-{nf_sym}-{style_compact_nf}"

# Replace with:
nf_sym = ""  # Removed suffix
postscript_name = f"{font_config.family_name_compact}-{style_compact_nf}"
```

**Change 2:** Update the `update_font_names()` call with clean names

```python
# Original:
update_font_names(
    font=nf_font,
    family_name=f"{font_config.family_name} {nf_sym}{style_nf_with_prefix_space}",
    style_name=style_in_2,
    full_name=f"{font_config.family_name} {nf_sym} {style_in_17}",
    version_str=font_config.version_str,
    postscript_name=postscript_name,
    unique_identifier=get_unique_identifier(
        font_config=font_config,
        postscript_name=postscript_name,
    ),
    is_skip_subfamily=is_skip_sufamily,
    preferred_family_name=f"{font_config.family_name} {nf_sym}",
    preferred_style_name=style_in_17,
)

# Replace with:
update_font_names(
    font=nf_font,
    family_name=font_config.family_name + style_nf_with_prefix_space,
    style_name=style_in_2,
    full_name=f"{font_config.family_name} {style_in_17}",
    version_str=font_config.version_str,
    postscript_name=postscript_name,
    unique_identifier=get_unique_identifier(
        font_config=font_config,
        postscript_name=postscript_name,
    ),
    is_skip_subfamily=is_skip_sufamily,
    preferred_family_name=font_config.family_name,
    preferred_style_name=style_in_17,
)
```

### Step 2: Modify Chinese font naming in `build.py`

Find the `build_cn()` function around line 1300. Make these changes:

**Change 1:** Simplify PostScript name (remove CN suffix)

```python
# Original:
postscript_name = f"{font_config.family_name_compact}-{build_option.cn_suffix_compact}-{style_compact_cn}"

# Replace with:
postscript_name = f"{font_config.family_name_compact}-{style_compact_cn}"
```

**Change 2:** Update the `update_font_names()` call with clean names

```python
# Original:
update_font_names(
    font=cn_font,
    family_name=f"{font_config.family_name} {build_option.cn_suffix}{style_cn_with_prefix_space}",
    style_name=style_in_2,
    full_name=f"{font_config.family_name} {build_option.cn_suffix} {style_in_17}",
    version_str=font_config.version_str,
    postscript_name=postscript_name,
    unique_identifier=get_unique_identifier(
        font_config=font_config,
        postscript_name=postscript_name,
        narrow=font_config.cn["narrow"],
    ),
    is_skip_subfamily=is_skip_subfamily,
    preferred_family_name=f"{font_config.family_name} {build_option.cn_suffix}",
    preferred_style_name=style_in_17,
)

# Replace with:
update_font_names(
    font=cn_font,
    family_name=font_config.family_name + style_cn_with_prefix_space,
    style_name=style_in_2,
    full_name=f"{font_config.family_name} {style_in_17}",
    version_str=font_config.version_str,
    postscript_name=postscript_name,
    unique_identifier=get_unique_identifier(
        font_config=font_config,
        postscript_name=postscript_name,
        narrow=font_config.cn["narrow"],
    ),
    is_skip_subfamily=is_skip_subfamily,
    preferred_family_name=font_config.family_name,
    preferred_style_name=style_in_17,
)
```

### Step 3: Modify Nerd Font base font naming in `nerdfont.py`

Find the `build_nf()` function around line 119. Make these changes:

**Change 1:** Update font names in the function

```python
# Original:
# Set font names
full_family_name = f"{family_name} NF Base{f' {suffix}' if suffix else ''}"
set_font_name(nf_font, full_family_name, 1)
set_font_name(nf_font, style_name, 2)
set_font_name(nf_font, f"{full_family_name} {style_name}", 4)
set_font_name(
    nf_font,
    f"{family_name.replace(' ', '-')}-NF-Base{f'-{suffix}' if suffix else ''}-{style_name}",
    6,
)

# Replace with:
# Set font names - using clean family name without "NF Base" suffix
base_suffix = f" {suffix}" if suffix else ""
full_family_name = f"{family_name}{base_suffix}"
set_font_name(nf_font, full_family_name, 1)
set_font_name(nf_font, style_name, 2)
set_font_name(nf_font, f"{full_family_name} {style_name}", 4)
set_font_name(
    nf_font,
    f"{family_name.replace(' ', '-')}{f'-{suffix}' if suffix else ''}-{style_name}",
    6,
)
```

**Change 2:** Update the output file path in `subset()` function around line 155

```python
# Original:
suffix = get_font_suffix(mono, propo)
_path = f"source/MapleMono-NF-Base{f'-{suffix}' if suffix else ''}.ttf"

# Replace with:
suffix = get_font_suffix(mono, propo)
_path = f"source/MapleMono{f'-{suffix}' if suffix else ''}.ttf"
```

## Result

After these changes, all generated fonts will use clean names:

### Before (with suffixes):
- Font family: "Maple Mono NF CN"
- Font full name: "Maple Mono NF CN Bold"
- PostScript name: "MapleMonoPro-NF-CN-Bold"

### After (without suffixes):
- Font family: "Maple Mono" 
- Font full name: "Maple Mono Bold"
- PostScript name: "MapleMonoPro-Bold"

## Testing

1. Modify your `config.json` to set the desired font name:
   ```json
   {
     "family_name": "Your Custom Font Name",
     "nerd_font": {
       "enable": true
     },
     "cn": {
       "enable": true
     }
   }
   ```

2. Run the build script:
   ```bash
   python build.py --cn --nf
   ```

3. Check the generated fonts - they should use exactly the name you specified without any NF/CN suffixes.

## Notes

- These changes affect both regular and variable font builds
- The modifications preserve all font functionality while only changing the naming
- Style suffixes (Bold, Italic, etc.) are still properly appended
- All font metadata (NameID 1, 4, 6, 16) will use the clean names
- Intermediate/base fonts (used during build) also follow the clean naming convention

## Backup

Always backup your original files before making changes:
```bash
cp build.py build.py.backup
cp source/py/task/nerdfont.py source/py/task/nerdfont.py.backup
```

This way you can easily restore the original functionality if needed.
