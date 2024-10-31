# Show-Hide-Tabs-and-Fields-based-on-Drop-Down

A custom JavaScript script to dynamically manage the visibility of elements in AEM component dialogs based on the selected value in a `select` field.

## Configuration and Usage

### 1. Create the Client Library for Dialogs

Navigate to the folder where client libraries are stored, typically at:
```
ui.apps/src/main/content/jcr_root/apps/<your-project>/clientlibs
```

Create a new folder for the dialog-specific client library, for example:
```
clientlib-dialog
```

### 2. Configure the Client Library

Inside `clientlib-dialog`, create `.content.xml` and `js.txt` files.

#### `.content.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          jcr:primaryType="cq:ClientLibraryFolder"
          categories="[cq.authoring.dialog]"/>
```

#### `js.txt`
List the JavaScript file name for loading in AEM:
```
dialog-visibility.js
```

### 3. Add the Custom JavaScript

Create `dialog-visibility.js` inside `clientlib-dialog` and paste the following code:

```javascript
(function(document, $) {
    "use strict";

    // Function to show or hide elements based on the selected value within the current dialog
    const toggleVisibility = (value, dialog) => {
        $(dialog).find(".show-hide-dialog").hide();

        const targetElements = $(dialog).find(`.show-hide-dialog.${value}`);
        targetElements.show();
    };

    // Function to handle visibility on the initial load based on the selected item
    const loadInitialVisibility = (dialog) => {
        const selectParent = dialog.querySelector('.show-hide-dialog-select');
        if (selectParent) {
            const selectedItem = getSelectedItem(selectParent);
            if (selectedItem) {
                const initialValue = selectedItem.getAttribute("data-value");
                toggleVisibility(initialValue, dialog);
            } else {
                console.error("⚠️ [Error] - No initially selected item found in the dialog.");
            }
        } else {
            console.error("❌ [Error] - 'show-hide-dialog-select' span not found in this dialog.");
        }
    };

    // Helper function to retrieve the selected item with a slight delay
    const getSelectedItem = (selectParent) => {
        return selectParent.querySelector('.coral-SelectList-item[aria-selected="true"]');
    };

    // Helper function to add classes to the corresponding tab
    const addClassesToTab = (additionalClasses, correspondingTab) => {
        additionalClasses.forEach(cls => correspondingTab.classList.add(cls));
    };

    // Function to retrieve and filter classes for adding to the tab
    const getAdditionalClasses = (element) => {
        return Array.from(element.classList)
            .filter(cls => cls === "show-hide-dialog" || (cls !== "foundation-layout-util-vmargin" && cls !== "show-hide-dialog"));
    };

    // Function to associate 'show-hide-dialog' class with the corresponding <coral-tab>, using setTimeout for delayed loading
    const associateClassWithTab = (dialog, callback) => {
        setTimeout(() => {
            const showHideElements = dialog.querySelectorAll('.show-hide-dialog.foundation-layout-util-vmargin');

            showHideElements.forEach((element) => {
                const coralPanelContent = element.closest('coral-panel-content');
                if (!coralPanelContent) return;

                const coralPanel = coralPanelContent.closest('._coral-Panel');
                if (coralPanel && coralPanel.hasAttribute('aria-labelledby')) {
                    const ariaLabelledbyId = coralPanel.getAttribute('aria-labelledby');
                    const correspondingTab = document.getElementById(ariaLabelledbyId);

                    if (correspondingTab) {
                        const additionalClasses = getAdditionalClasses(element);
                        addClassesToTab(additionalClasses, correspondingTab);
                    } else {
                        console.error(`⚠️ [Error] - No <coral-tab> found with ID: ${ariaLabelledbyId}`);
                    }
                } else {
                    console.error("⚠️ [Error] - '_coral-Panel' parent or 'aria-labelledby' not found on element.");
                }
            });

            if (callback) callback();
        }, 25); // Adjust delay as necessary
    };

    $(document).on("dialog-ready", function() {
        const dialog = this;

        associateClassWithTab(dialog, () => loadInitialVisibility(dialog));

        const selectParent = dialog.querySelector('.show-hide-dialog-select');
        if (selectParent) {
            const selectListItems = selectParent.querySelectorAll('.coral-SelectList-item');

            selectListItems.forEach((item) => {
                item.addEventListener("click", function() {
                    setTimeout(() => {
                        const selectedItem = getSelectedItem(selectParent);
                        if (selectedItem) {
                            const newValue = selectedItem.getAttribute("data-value");
                            toggleVisibility(newValue, dialog);
                        } else {
                            console.error("⚠️ [Error] - No selected item found.");
                        }
                    }, 25); // Adjust delay as necessary
                });
            });
        } else {
            console.error("❌ [Error] - 'show-hide-dialog-select' span not found in this dialog.");
        }
    });
})(document, Granite.$);
```

## Class Usage for Visibility Control

### Select Field to Control Visibility

Ensure the select field in the dialog has the class `show-hide-dialog-select`. Example:

```xml
<select jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/foundation/form/select"
        class="show-hide-dialog-select"
        fieldLabel="Variant" name="./variant">
    <items jcr:primaryType="nt:unstructured">
        <solid jcr:primaryType="nt:unstructured" text="Solid Background" value="solid" selected="true" />
        <image jcr:primaryType="nt:unstructured" text="Image Background" value="image" />
    </items>
</select>
```

### Conditional Dialog Elements or Tabs

Elements or tabs should have the `show-hide-dialog` class followed by the select value to determine their visibility. Example:

```xml
<solid jcr:primaryType="nt:unstructured"
       sling:resourceType="granite/ui/components/foundation/container"
       class="show-hide-dialog solid">
    <items jcr:primaryType="nt:unstructured">
        <title
            jcr:primaryType="nt:unstructured"
            sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
            fieldLabel="Title"
            name="./title"/>
    </items>
</solid>
```

For a conditional tab:

```xml
<media
        jcr:primaryType="nt:unstructured"
        jcr:title="Media"
        sling:resourceType="granite/ui/components/coral/foundation/container"
        granite:class="show-hide-dialog image"
        margin="{Boolean}true">
    <items jcr:primaryType="nt:unstructured">
        <columns
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/fixedcolumns"
                margin="{Boolean}true">
            <items jcr:primaryType="nt:unstructured">
                <column
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/container">
                    <items jcr:primaryType="nt:unstructured">
                      <file
                              granite:class="cmp-image__editor-file-upload"
                              jcr:primaryType="nt:unstructured"
                              sling:resourceType="cq/gui/components/authoring/dialog/fileupload"
                              fieldLabel="Asset Generic"
                              required="true"
                              class="cq-droptarget"
                              enableNextGenDynamicMedia="{Boolean}true"
                              fileNameParameter="./fileName"
                              fileReferenceParameter="./fileReference"
                              damOnly="{Boolean}true"
                              rootPath="/content/dam"
                              mimeTypes="[image/gif,image/jpeg,image/png,image/tiff,image/svg+xml,image/webp,video/mp4]"
                              name="./file"/>
                    </items>
                </column>
            </items>
        </columns>
    </items>
</media>
```
