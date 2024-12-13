# Customize your TaxHub experience

Customize your TaxHub for a seamless integration with your application. From colors and fonts to border-radius, our flexible style options allow you to match the design of your site with the appearance option.

1. **Start by picking a theme.**
   Get up and running right away by picking the prebuilt theme that most closely resembles your platform.

2. **Customize the theme using variables .**
   Set variables like fontFamily and brand color to broadly customize components appearing throughout each TaxHub.

3. **Reach out for more options.**
   If you're stuck or have any requests - just reach out to our team and we can help you create and manage your TaxHub styling.

# Themes

Start customizing TaxHub by picking from one of the following themes:

-   `Cobalt`
-   `Purple`
-   `Blue Ribbon`
-   `Black Pink`

# Variables

Set variables to affect the appearance of many components appearing throughout each TaxHub.

The variables option works like CSS variables.

```
const appearance = {
  theme: 'cobalt',

  variables: {
    colorPrimary: '#0570be',
    colorBackground: '#fffff0',
    colorText: '#30213d',
    colorDanger: '#dfb41',
    fontFamily: 'Roboto, system-ui, sans-serif',
    spacingUnit: '2px',
    borderRadius: '4px',
    // See all possible variables below
  }
};
```

# Typography

## Primary

This is the primary font used for text to display to the user. We recommend you use a sans-serif font as they tend to be more ledgible.

## Secondary

This is the secondary font used for displaying numbers within the app. This can be the same as the primary font.

# Styles

If you have a special font, you can specify the following properties for each typography style:

| Style                  | Description                                        |
| ---------------------- | -------------------------------------------------- |
| h1                     | Main heading style, typically used for page titles |
| h2                     | Secondary heading style, used for major sections   |
| h3                     | Tertiary heading style, used for subsections       |
| h4                     | Fourth-level heading style                         |
| h5                     | Fifth-level heading style                          |
| body/bold              | Bold variant of the main body text                 |
| body/light             | Light weight variant of the main body text         |
| body/regular           | Regular weight body text, used for most content    |
| caption/large/bold     | Large bold caption text                            |
| caption/large/regular  | Large regular weight caption text                  |
| caption/medium/bold    | Medium-sized bold caption text                     |
| caption/medium/regular | Medium-sized regular weight caption text           |
| caption/small/bold     | Small bold caption text                            |
| caption/small/regular  | Small regular weight caption text                  |

For each of these styles, you can customize:

-   `fontWeight`: The thickness of the characters
-   `fontStyle`: Normal, italic, or oblique
-   `letterSpacing`: Space between characters
-   `lineHeight`: Space between lines of text
-   `textDecoration`: Underline, overline, or line-through effects

# Colors

We offer plenty of customization here. If you are unsure please reach out to CTC and we can help you decide which colors to use here.

## Background Colors

| Token    | Description                                           |
| -------- | ----------------------------------------------------- |
| Brand    | Primary brand color used for main UI elements         |
| Danger   | Used for error states and destructive actions         |
| Disabled | Applied to inactive or disabled elements              |
| Input    | Used for form input backgrounds                       |
| Neutral  | Default background color for non-interactive elements |
| Overlay  | Used for modal backgrounds and overlays               |
| Success  | Indicates successful actions or positive states       |
| Warning  | Used for warning messages and cautionary states       |

### Accent Colors

| Token   | Description                                        |
| ------- | -------------------------------------------------- |
| Green   | Used for success states and positive indicators    |
| Neutral | Used for non-interactive or secondary elements     |
| Orange  | Used for warning states and medium-priority alerts |
| Red     | Used for error states and critical alerts          |
| Brand   | Variations of the primary brand color              |
| Yellow  | Used for cautionary states and low-priority alerts |

## Border Colors

| Token    | Description                                  |
| -------- | -------------------------------------------- |
| Brand    | Primary border color matching brand identity |
| Danger   | Border color for error states                |
| Disabled | Border color for inactive elements           |
| Success  | Border color for success states              |
| Warning  | Border color for warning states              |
| Neutral  | Default border color for general use         |

## Button Colors

| Token    | Description                                             |
| -------- | ------------------------------------------------------- |
| Brand    | Primary button color matching brand identity            |
| Danger   | Used for destructive or irreversible actions            |
| Neutral  | Default button color for general actions                |
| Subtle   | Low-emphasis buttons for secondary actions              |
| Success  | Buttons indicating positive or successful actions       |
| Warning  | Buttons for actions requiring user attention            |
| Disabled | Style applied to buttons that cannot be interacted with |

_Note: Each button color (except Disabled) supports `default`, `hover`, and `pressed` states._

## Text Colors

| Token    | Description                                    |
| -------- | ---------------------------------------------- |
| Brand    | Text color matching brand identity             |
| Danger   | Text color for error messages and warnings     |
| Default  | Primary text color for general content         |
| Disabled | Text color for non-interactive elements        |
| Low      | Subdued text color for secondary content       |
| High     | High-contrast text color for important content |
| Inverse  | Text color for use on dark backgrounds         |
| Success  | Text color for success messages                |
| Warning  | Text color for warning messages                |

## Icon Colors

| Token    | Description                               |
| -------- | ----------------------------------------- |
| Brand    | Icon color matching brand identity        |
| Danger   | Icon color for error states and warnings  |
| Default  | Primary icon color for general use        |
| Disabled | Icon color for non-interactive elements   |
| Low      | Subdued icon color for secondary elements |
| High     | High-contrast icon color for emphasis     |
| Inverse  | Icon color for use on dark backgrounds    |
| Success  | Icon color for success states             |
| Warning  | Icon color for warning states             |

## Border Radius

For the primary button and measured in pixels. `0` means no border radius, and `999` means its completely rounded. Default value is `4`.

# Help

If you're stuck or have any requests - just reach out to our team and we can help you create and manage your TaxHub styling.
