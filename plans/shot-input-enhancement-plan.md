# Shot Distance Input Enhancement Plan

## Problem Statement
The current implementation uses standard `<input type="number">` elements for entering shot distances (0-600 yards/feet). On mobile devices, the native numeric keyboard appears and obscures a significant portion of the screen, making it difficult for players to see the shot entry context while inputting values.

## Solution Overview
Replace the native numeric keyboard with a custom bottom-sheet numeric pad that:
- Slides up from the bottom of the screen
- Displays large, touch-friendly number buttons (0-9)
- Shows the current input value prominently
- Provides backspace and clear functionality
- Does NOT include quick preset buttons (user preference)
- Keeps the shot entry form visible while entering numbers

## Design Specifications

### UI/UX Design

#### Layout Structure
```
┌─────────────────────────────────┐
│      Shot Entry Form            │
│      (Fully Visible)            │
│                                 │
│  Start Distance: [TAP TO OPEN] │
│  End Distance:   [TAP TO OPEN] │
│                                 │
└─────────────────────────────────┘
         ↓ (tap input) ↓
┌─────────────────────────────────┐
│  Current Value: 150             │
│  ┌───┬───┬───┐                 │
│  │ 1 │ 2 │ 3 │                 │
│  ├───┼───┼───┤                 │
│  │ 4 │ 5 │ 6 │                 │
│  ├───┼───┼───┤                 │
│  │ 7 │ 8 │ 9 │                 │
│  ├───┼───┼───┤                 │
│  │ ⌫ │ 0 │ ✓ │   (⌫=backspace) │
│  └───┴───┴───┘                 │
│         [Done]                  │
└─────────────────────────────────┘
```

#### Visual Design
- **Background**: Semi-transparent overlay (#000000 at 40% opacity) behind the pad
- **Pad Background**: White (#FFFFFF) with rounded top corners (16px radius)
- **Button Grid**: 3 columns, equal width buttons
- **Number Buttons**: 
  - Size: Minimum 60px height, full column width
  - Background: #F5F4F0 (light gray)
  - Active state: #E8E6E0 (darker gray)
  - Font: 24px, Inter font, #2C3333 color
  - Border radius: 8px
  - Margin between buttons: 8px
- **Value Display**:
  - Large text: 48px, bold, #2D5016 (green)
  - Positioned above the button grid
  - Shows unit suffix (yd/ft) based on lie type
- **Action Buttons**:
  - Backspace (⌫): Uses existing accent color (#6B7280)
  - Done/Confirm (✓): Uses primary green (#2D5016), filled button
- **Done Button**:
  - Full width at bottom
  - Primary green background
  - White text, 16px

#### Animations
- **Open**: Slide up from bottom with 300ms ease-out
- **Close**: Slide down with 250ms ease-in
- **Button Press**: Scale to 0.95 with 100ms transition

### Component Architecture

#### New Components to Create

1. **NumericPad (Modal Component)**
   - Props:
     - `isOpen`: boolean - controls visibility
     - `value`: string - current input value
     - `unit`: string - 'yd' or 'ft'
     - `onChange`: function(value) - called when value changes
     - `onClose`: function - called when pad should close
     - `onDone`: function - called when user confirms value
     - `minValue`: number (default: 0)
     - `maxValue`: number (default: 600)
   - Internal state:
     - `displayValue`: string - value being built

2. **Modified Input Fields**
   - Replace `type="number"` inputs with custom touchable divs
   - Display current value and placeholder
   - On tap, open the numeric pad
   - Maintain existing disabled/readonly behavior

### Implementation Details

#### CSS Additions (in index.html style section)
```css
/* Numeric Pad Overlay */
.numeric-pad-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.4);
    z-index: 1000;
    display: flex;
    align-items: flex-end;
    justify-content: center;
}

.numeric-pad-overlay.hidden {
    display: none;
}

/* Numeric Pad Container */
.numeric-pad {
    background: white;
    width: 100%;
    max-width: 400px;
    border-radius: 16px 16px 0 0;
    padding: 1rem;
    padding-bottom: max(1rem, env(safe-area-inset-bottom));
    animation: slideUp 300ms ease-out;
}

@keyframes slideUp {
    from { transform: translateY(100%); }
    to { transform: translateY(0); }
}

/* Value Display */
.numeric-pad-value {
    text-align: center;
    font-size: 48px;
    font-weight: 700;
    color: #2D5016;
    padding: 0.5rem 0 1rem;
    font-family: 'Inter', sans-serif;
}

.numeric-pad-value .unit {
    font-size: 24px;
    color: #6B7280;
    margin-left: 4px;
}

/* Button Grid */
.numeric-pad-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;
    margin-bottom: 1rem;
}

/* Number Buttons */
.numeric-pad-btn {
    height: 60px;
    background: #F5F4F0;
    border: none;
    border-radius: 8px;
    font-size: 24px;
    font-weight: 500;
    color: #2C3333;
    cursor: pointer;
    transition: transform 100ms, background 100ms;
    -webkit-tap-highlight-color: transparent;
}

.numeric-pad-btn:active {
    transform: scale(0.95);
    background: #E8E6E0;
}

/* Special Buttons */
.numeric-pad-btn.backspace {
    background: #F5F4F0;
    font-size: 20px;
}

.numeric-pad-btn.done {
    background: #2D5016;
    color: white;
    font-size: 16px;
    font-weight: 600;
}

/* Done Button */
.numeric-pad-done {
    width: 100%;
    padding: 1rem;
    background: #2D5016;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    -webkit-tap-highlight-color: transparent;
}

.numeric-pad-done:active {
    background: #3D6B22;
}

/* Touch Input Display (replaces standard input) */
.form-input-touchable {
    width: 100%;
    padding: 0.75rem 0.875rem;
    font-family: 'Inter', sans-serif;
    font-size: 16px;
    background: white;
    border: 1.5px solid #E8E6E0;
    border-radius: 4px;
    color: #2C3333;
    cursor: pointer;
    min-height: 48px;
    display: flex;
    align-items: center;
}

.form-input-touchable:focus {
    outline: none;
    border-color: #3D6B22;
}

.form-input-touchable.placeholder {
    color: #9CA3AF;
}

.form-input-touchable.disabled {
    background: #F5F4F0;
    color: #6B7280;
    cursor: not-allowed;
}
```

#### React Component Logic

```javascript
// NumericPad Component
function NumericPad({ isOpen, value, unit, onChange, onClose, onDone, minValue = 0, maxValue = 600 }) {
    const [displayValue, setDisplayValue] = useState(value || '');
    
    useEffect(() => {
        setDisplayValue(value || '');
    }, [value, isOpen]);
    
    const handleNumber = (num) => {
        let newValue = displayValue + num;
        // Validate against max
        if (parseInt(newValue) > maxValue) {
            newValue = maxValue.toString();
        }
        setDisplayValue(newValue);
        onChange(newValue);
    };
    
    const handleBackspace = () => {
        const newValue = displayValue.slice(0, -1);
        setDisplayValue(newValue);
        onChange(newValue);
    };
    
    const handleClear = () => {
        setDisplayValue('');
        onChange('');
    };
    
    const handleDone = () => {
        onDone(displayValue);
    };
    
    if (!isOpen) return null;
    
    return (
        <div className="numeric-pad-overlay" onClick={onClose}>
            <div className="numeric-pad" onClick={e => e.stopPropagation()}>
                <div className="numeric-pad-value">
                    {displayValue || '0'}<span className="unit">{unit}</span>
                </div>
                
                <div className="numeric-pad-grid">
                    {[1,2,3,4,5,6,7,8,9].map(num => (
                        <button 
                            key={num}
                            className="numeric-pad-btn"
                            onClick={() => handleNumber(num.toString())}
                        >
                            {num}
                        </button>
                    ))}
                    <button 
                        className="numeric-pad-btn backspace"
                        onClick={handleBackspace}
                    >
                        ⌫
                    </button>
                    <button 
                        className="numeric-pad-btn"
                        onClick={() => handleNumber('0')}
                    >
                        0
                    </button>
                    <button 
                        className="numeric-pad-btn done"
                        onClick={handleDone}
                    >
                        ✓
                    </button>
                </div>
                
                <button className="numeric-pad-done" onClick={handleDone}>
                    Done
                </button>
            </div>
        </div>
    );
}
```

### User Experience Flow

1. **Viewing Shot Entry Form**
   - User sees the full shot entry form with start/end distances
   - Distance fields show current values or placeholders
   - Previous shot banner remains visible

2. **Opening Numeric Pad**
   - User taps on a distance input field
   - Bottom sheet slides up with numeric pad
   - Current value is pre-populated if exists
   - User can still see the form above the pad

3. **Entering Values**
   - User taps number buttons to enter digits
   - Value updates in real-time in the display
   - Max value (600) is enforced
   - User can tap backspace to delete last digit

4. **Confirming Value**
   - User taps "Done" button or the ✓ button
   - Numeric pad slides down
   - Selected value appears in the input field
   - Form context remains visible

5. **Canceling Input**
   - User taps outside the pad (on overlay)
   - Pad closes without saving changes
   - Original value is preserved

### Edge Cases to Handle

1. **Empty Input**: Allow empty (cleared) values, treat as invalid until filled
2. **Single Zero**: Allow "0" as valid input (for holed putts)
3. **Rapid Tapping**: Debounce not needed for button clicks (synchronous)
4. **Landscape Mode**: Ensure pad fits properly in landscape orientation
5. **Accessibility**: Add aria-labels to buttons for screen readers

### Files to Modify

1. **index.html** - Main implementation file (single-file React app)
   - Add new CSS styles (lines ~518)
   - Add NumericPad component (within the script babel section)
   - Modify GolfTracker component to use touchable inputs and NumericPad
   - Replace `ref` and `onChange` logic for distance inputs

## Implementation Checklist

- [ ] Add CSS styles for numeric pad overlay, container, buttons
- [ ] Add CSS for touchable input fields (replacing standard inputs)
- [ ] Create NumericPad React component
- [ ] Add state for pad visibility and active field
- [ ] Replace startDistance input with touchable div + NumericPad
- [ ] Replace endDistance input with touchable div + NumericPad
- [ ] Wire up onDone handlers to update currentShot state
- [ ] Handle unit display (yd/ft) based on lie type
- [ ] Test keyboard does NOT appear on input tap
- [ ] Verify form remains visible while using pad
- [ ] Test value persistence and validation
- [ ] Ensure disabled state still works for startDistance on subsequent shots
