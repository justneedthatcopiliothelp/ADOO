<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>A Dark Ocean</title>
    <style>
        /* Define a consistent, monochromatic color palette */
        :root {
            --color-bg: #fff;
            --color-fg: #000;
            --color-accent: #ddd;
            --color-button-bg: #fff;
            --color-button-hover: #eee;
            --color-border: #000;
        }

        /* Basic body styling for full-screen monochromatic feel */
        body {
            font-family: 'Courier New', Courier, monospace;
            background-color: var(--color-bg);
            color: var(--color-fg);
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            padding: 2rem;
            box-sizing: border-box;
            perspective: 1000px;
        }
        
        /* The main container for the game, now with a transform for the swaying effect */
        .game-container-wrapper {
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: transform 0.2s ease-out; /* Smooth transition for the sway */
        }
        
        /* Camera shake effect */
        @keyframes cameraShake {
            0%, 100% { transform: rotate(0) translate(0); }
            10% { transform: rotate(-1deg) translate(-2px, -2px); }
            20% { transform: rotate(1deg) translate(2px, 2px); }
            30% { transform: rotate(-1deg) translate(-2px, -2px); }
            40% { transform: rotate(1deg) translate(2px, 2px); }
            50% { transform: rotate(-1deg) translate(-2px, -2px); }
            60% { transform: rotate(1deg) translate(2px, 2px); }
            70% { transform: rotate(-1deg) translate(-2px, -2px); }
            80% { transform: rotate(1deg) translate(2px, 2px); }
            90% { transform: rotate(-1deg) translate(-2px, -2px); }
        }

        .camera-shake {
            animation: cameraShake 0.5s ease-in-out forwards;
        }

        /* Main game container with flex layout for side-by-side panels */
        .game-container {
            display: flex;
            flex-direction: row;
            width: 100%;
            max-width: 1200px;
            height: 90vh;
        }
        
        /* New screen for the swimming loading animation */
        .swim-loading-screen {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #fff;
            color: #000;
            z-index: 20;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        /* Styling for the main game area (left panel) */
        .game-main-area {
            flex: 2;
            display: flex;
            flex-direction: column;
            padding: 2rem;
            border-right: 2px solid var(--color-border);
            position: relative;
        }
        
        /* New styling for the navigation tabs */
        .navigation-tabs {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            display: flex;
            justify-content: flex-start;
            border-bottom: 2px solid var(--color-border);
        }
        
        .tab-button {
            padding: 1rem 2rem;
            font-size: 1.25rem;
            font-weight: bold;
            color: var(--color-fg);
            background-color: var(--color-bg);
            border: none;
            border-right: 2px solid var(--color-border);
            cursor: pointer;
            text-transform: uppercase;
            outline: none;
            position: relative;
        }
        
        /* Style for the new item indicator */
        .new-indicator {
            position: absolute;
            top: 5px;
            right: 5px;
            width: 10px;
            height: 10px;
            background-color: #f00;
            border-radius: 50%;
            animation: pulse 1s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.5); }
            100% { transform: scale(1); }
        }

        .tab-button.active {
            background-color: var(--color-accent);
        }

        /* Styling for the narrative display panel (top-left) */
        .narrative-panel {
            position: absolute;
            top: 4rem; /* Adjusted to accommodate the new nav bar */
            left: 2rem;
            right: 2rem;
            line-height: 1.6;
            font-size: 0.9em; /* Smaller font size as requested */
            overflow-y: auto;
            max-height: calc(100% - 6rem); /* Adjusted to give space for the button at the bottom */
        }
        
        /* Container for the narrative lines with scroll logic */
        .narrative-lines-container {
            position: relative;
        }

        /* Styling for the narrative text lines with fade-in/out effect */
        .narrative-line {
            opacity: 0;
            transform: translateY(10px);
            transition: opacity 1s ease-in, transform 1s ease-in;
            margin: 0;
            padding: 0;
        }
        
        .narrative-line.fade-out {
            opacity: 0;
            height: 0;
            margin-top: -1.6em; /* Adjust to match line-height */
            overflow: hidden;
            transition: opacity 1s, height 1s, margin 1s;
        }

        /* The main interactive button */
        .game-button {
            /* Positioning is now handled by a dedicated container */
            padding: 0.75rem 1.5rem; /* Smaller padding */
            font-size: 1rem; /* Smaller font size */
            font-weight: bold;
            color: var(--color-fg);
            background-color: var(--color-button-bg);
            border: 2px solid var(--color-border);
            cursor: pointer;
            transition: background-color 0.2s;
            text-transform: uppercase;
            
            /* Styles for the cooldown bar animation */
            position: relative;
            overflow: hidden;
        }

        /* New container for the main button to position it at the bottom */
        .main-button-container {
            position: absolute;
            bottom: 2rem; /* Position the button 2rem from the bottom of its parent */
            left: 50%;
            transform: translateX(-50%); /* Center the button horizontally */
        }
        
        /* Cooldown bar inside the button */
        .cooldown-bar {
            position: absolute;
            bottom: 0;
            left: 0;
            height: 5px;
            background-color: var(--color-fg);
            width: 0;
            transition: width 0.1s linear; /* for smooth updates */
        }

        .game-button.disabled {
            cursor: not-allowed;
            background-color: #eee;
            color: #888;
            border-color: #888;
        }
        
        .game-button.stunned-button {
            background-color: #555 !important;
            color: #aaa !important;
            cursor: not-allowed !important;
        }


        /* Hover effect for the button */
        .game-button:hover:not(.disabled) {
            background-color: var(--color-button-hover);
        }

        /* Styling for the inventory panel (right side) */
        .inventory-panel {
            flex: 1;
            padding: 1rem;
            display: none; /* Hidden by default */
            flex-direction: column;
            align-items: center;
            justify-content: center;
            border-left: 2px solid var(--color-border);
            max-width: 300px; /* Make the inventory panel a bit smaller */
        }

        .inventory-box {
            width: 100%;
            height: 100%;
            border: 2px solid var(--color-border);
            padding: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 1rem;
            overflow-y: auto;
        }

        .inventory-item {
            padding: 0.5rem 1rem;
            border: 1px solid var(--color-border);
            font-size: 1rem;
            display: flex;
            justify-content: space-between;
            width: 100%;
        }

        .inventory-item.clickable {
            cursor: pointer;
        }
        
        .inventory-item.selected {
            background-color: var(--color-accent);
        }
        
        .inventory-item.disabled {
            color: #888;
            border-color: #888;
            cursor: not-allowed;
        }
        
        /* New styling for the resource gathering area page */
        .resource-gathering-page {
            display: none;
            width: 100%;
            height: 100%;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            padding: 2rem;
            text-align: center;
            gap: 1rem; /* Added gap for the new swim button */
        }

        .crafting-recipe {
            border: 2px solid var(--color-border);
            padding: 1.6rem; /* 20% smaller */
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 0.8rem; /* 20% smaller */
            width: 100%; /* Ensure it takes up container width */
            box-sizing: border-box;
        }
        
        /* New styling for the craft area page */
        .craft-area-page {
            display: none; /* Hidden by default */
            width: 100%;
            height: 100%;
            justify-content: flex-start; /* Align to top */
            align-items: center;
            flex-direction: column;
            padding: 2rem;
            text-align: center;
            overflow-y: auto;
        }
        
        .crafting-recipes-container {
            width: 100%;
            display: grid; /* Use grid for 2-column layout */
            grid-template-columns: repeat(2, 1fr);
            gap: 1rem;
        }
        
        /* New styling for the achievement area page */
        .achievement-area-page {
            display: none; /* Hidden by default */
            width: 100%;
            height: 100%;
            justify-content: flex-start;
            align-items: center;
            flex-direction: column;
            padding: 2rem;
            overflow-y: auto;
        }

        /* Modal Popup styles */
        .modal {
            display: none; /* Hidden by default */
            position: fixed;
            z-index: 10;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.4);
            backdrop-filter: blur(5px);
            justify-content: center;
            align-items: center;
        }

        .modal-content {
            background-color: var(--color-bg);
            border: 2px solid var(--color-border);
            padding: 2rem;
            width: 80%;
            max-width: 400px;
            text-align: center;
        }

        .modal-button {
            margin-top: 1rem;
            padding: 0.75rem 1.5rem;
            font-size: 1rem;
            font-weight: bold;
            color: var(--color-fg);
            background-color: var(--color-button-bg);
            border: 2px solid var(--color-border);
            cursor: pointer;
        }
        
        /* Mobile-first responsiveness */
        @media (max-width: 768px) {
            .game-container {
                flex-direction: column;
                height: 100vh;
            }
            .game-main-area {
                border-right: none;
                border-bottom: 2px solid var(--color-border);
                height: 60%;
                width: 100%;
            }
            .inventory-panel {
                height: 40%;
                width: 100%;
                border-left: none; /* remove left border on mobile */
                max-width: 100%;
            }
            .main-button-container {
                bottom: 2rem;
                left: 50%;
                transform: translateX(-50%);
            }
            .navigation-tabs {
                justify-content: space-around;
                flex-wrap: wrap; /* Allow tabs to wrap on small screens */
            }
            .tab-button {
                padding: 0.5rem 1rem; /* Smaller padding on mobile */
                font-size: 1rem;
            }
            .crafting-recipes-container {
                grid-template-columns: 1fr; /* Single column on mobile */
            }
        }
        
        .music-control-button {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 5;
            padding: 10px;
            font-size: 1rem;
            font-weight: bold;
            color: var(--color-fg);
            background-color: var(--color-button-bg);
            border: 2px solid var(--color-border);
            cursor: pointer;
        }
        
        /* New achievement styles */
        .achievement-container {
            position: fixed;
            bottom: 20px;
            right: 20px;
            z-index: 10;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .achievement-popup {
            background-color: var(--color-fg);
            color: var(--color-bg);
            border: 2px solid var(--color-border);
            padding: 15px 20px;
            border-radius: 5px;
            text-align: left;
            opacity: 0;
            transform: translateY(20px);
            animation: fadeInOut 4s ease-in-out forwards;
        }

        @keyframes fadeInOut {
            0% { opacity: 0; transform: translateY(20px); }
            10% { opacity: 1; transform: translateY(0); }
            90% { opacity: 1; transform: translateY(0); }
            100% { opacity: 0; transform: translateY(20px); }
        }
        
        .achievement-title {
            font-size: 1.2rem;
            font-weight: bold;
        }

        .achievement-description {
            font-size: 0.9rem;
        }
        
        .achievement-rarity {
            font-size: 0.8rem;
            font-style: italic;
            color: #ccc;
        }

        .achievement-rarity-section {
            width: 100%;
            margin-bottom: 2rem;
        }

        .achievement-rarity-section h2 {
            border: 2px solid var(--color-border); /* Changed from color-coded to black */
            padding: 0.5rem;
            margin-bottom: 1rem;
            text-align: left;
        }

        .achievement-list-item {
            width: 100%;
            border: 2px solid var(--color-border);
            padding: 1rem;
            margin-bottom: 1rem;
            box-sizing: border-box;
        }
        
        .achievement-list-item.unlocked {
            background-color: var(--color-accent);
        }
        
        .back-button {
            margin-top: 2rem;
        }

        /* New CSS for the loading screen */
        .loading-screen {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100%;
            width: 100%;
        }

        .bouncing-text {
            font-size: 2rem;
            font-weight: bold;
            display: flex; /* To position letters side-by-side */
            gap: 0.2rem;
        }

        .bouncing-text span {
            display: inline-block;
            animation: bounce 1s infinite alternate;
        }

        @keyframes bounce {
            from {
                transform: translateY(0);
            }
            to {
                transform: translateY(-10px);
            }
        }

        .loading-bar-container {
            width: 80%;
            max-width: 400px;
            height: 5px;
            border: 2px solid var(--color-border);
            margin-top: 2rem;
            overflow: hidden;
        }

        .loading-bar {
            width: 0;
            height: 100%;
            background-color: var(--color-fg);
            transition: width 0.1s linear;
        }
        
        /* New CSS for the progression screen */
        .progression-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
            text-align: center;
        }
        
        .progression-graph {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 2rem;
            position: relative;
        }
        
        .progression-section {
            border: 2px solid var(--color-border);
            padding: 1rem 2rem;
            font-size: 1.25rem;
            font-weight: bold;
            text-transform: uppercase;
            text-align: center;
            min-width: 300px;
            position: relative;
        }
        
        .progression-section.locked {
            filter: blur(5px);
        }
        
        .connector-line {
            width: 2px;
            height: 2rem;
            background-color: var(--color-border);
        }
        
        /* New Workers screen styles */
        .workers-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            width: 100%;
            height: 100%;
            text-align: center;
            padding: 2rem;
        }
        
        .worker-assignment-container {
            display: flex;
            flex-direction: column;
            gap: 1rem;
            width: 100%;
            max-width: 400px;
        }
        
        .worker-stats {
            margin-bottom: 2rem;
            font-size: 1.2rem;
            font-weight: bold;
        }
        
        .assignment-option {
            border: 2px solid var(--color-border);
            padding: 1rem;
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
        }
        
        .assign-button {
            padding: 0.5rem 1rem;
            font-size: 0.9rem;
            font-weight: bold;
            color: var(--color-fg);
            background-color: var(--color-button-bg);
            border: 1px solid var(--color-border);
            cursor: pointer;
        }
        
        /* Dive Screen and Mini-game styles */
        .dive-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
            padding: 2rem;
            gap: 2rem;
        }

        #dive-prep-area, #deep-dive-prep-area {
            text-align: center;
        }
        
        .tool-selection-container {
            display: flex;
            flex-direction: column;
            gap: 1rem;
            margin-top: 1rem;
            margin-bottom: 1rem;
        }
        
        .tool-selection-item {
            display: flex;
            align-items: center;
            gap: 1rem;
        }

        #dive-minigame-area, #deep-dive-minigame-area {
            display: none;
            flex-direction: column;
            align-items: center;
            gap: 1rem;
            width: 100%;
            height: 100%;
            overflow: hidden; /* Prevent internal scrollbars */
        }
        
        #minigame-grid, #deep-minigame-grid {
            font-family: 'Courier New', Courier, monospace;
            font-size: 1.2rem;
            line-height: 1.2rem;
            white-space: pre;
            border: 2px solid var(--color-border);
            padding: 1rem;
            width: 100%; /* Fill parent */
            height: 100%; /* Fill parent */
            flex-grow: 1; /* Take up available vertical space */
            overflow: auto;
            position: relative;
            box-sizing: border-box;
        }
        
        .player { color: blue; font-weight: bold; }
        .boat { color: green; font-weight: bold; }
        .landmark { color: red; font-weight: bold; cursor: help; position: relative; }
        .safe-path { color: #aaa; } /* New style for safe paths */
        .habitat { color: purple; font-weight: bold; } /* New style for habitats */
        .pet { font-size: 0.6rem; vertical-align: middle; animation: bob 1s infinite ease-in-out; }

        @keyframes bob {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-3px); }
        }


        /* Tooltip for landmarks */
        .landmark .tooltip-text {
            visibility: hidden;
            width: 120px;
            background-color: black;
            color: #fff;
            text-align: center;
            border-radius: 6px;
            padding: 5px 0;
            position: absolute;
            z-index: 1;
            bottom: 125%;
            left: 50%;
            margin-left: -60px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .landmark:hover .tooltip-text {
            visibility: visible;
            opacity: 1;
        }
        
        /* Commands Screen Styles */
        .commands-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
            text-align: center;
            gap: 1rem;
        }
        
        .admin-input {
            padding: 0.5rem;
            font-size: 1rem;
            border: 2px solid var(--color-border);
            width: 250px;
            text-align: center;
            margin-bottom: 1rem;
        }
        
        #admin-status {
            font-style: italic;
            height: 1.5rem; /* Reserve space to prevent layout shift */
        }
        
        #admin-controls-area {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
        }

        .toggle-switch {
            display: none; /* Hidden by default */
            align-items: center;
            justify-content: space-between;
            gap: 1rem;
            width: 100%;
        }

        .toggle-switch input {
            height: 0;
            width: 0;
            visibility: hidden;
        }

        .toggle-switch label {
            cursor: pointer;
            text-indent: -9999px;
            width: 50px;
            height: 25px;
            background: #aaa;
            display: block;
            border-radius: 25px;
            position: relative;
        }

        .toggle-switch label:after {
            content: '';
            position: absolute;
            top: 2.5px;
            left: 2.5px;
            width: 20px;
            height: 20px;
            background: #fff;
            border-radius: 20px;
            transition: 0.3s;
        }

        .toggle-switch input:checked + label {
            background: #008000;
        }

        .toggle-switch input:checked + label:after {
            left: calc(100% - 2.5px);
            transform: translateX(-100%);
        }
        
        /* Combat Modal Styles */
        #combat-modal .modal-content {
            max-width: 600px;
        }

        .combat-arena {
            display: flex;
            justify-content: space-around;
            align-items: center;
            margin-bottom: 1rem;
        }

        .combatant {
            text-align: center;
        }

        .combatant-char {
            font-size: 3rem;
            font-weight: bold;
        }

        .health-bar-container {
            width: 150px;
            height: 10px;
            border: 1px solid var(--color-border);
            margin-top: 0.5rem;
        }

        .health-bar {
            height: 100%;
            background-color: red;
            width: 100%;
            transition: width 0.2s;
        }
        
        .combat-actions {
            margin-top: 1rem;
            display: flex;
            gap: 0.5rem;
            justify-content: center;
            flex-wrap: wrap;
        }
        
        #combat-log {
            margin-top: 1rem;
            height: 60px;
            overflow-y: auto;
            border: 1px solid var(--color-accent);
            padding: 0.5rem;
            text-align: left;
        }
        
        /* Save/Load Screen Styles */
        .save-load-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
            text-align: center;
            gap: 1.5rem;
        }
        
        .save-load-section {
            width: 80%;
            max-width: 500px;
        }
        
        .save-load-textarea {
            width: 100%;
            height: 150px;
            padding: 0.5rem;
            font-family: 'Courier New', Courier, monospace;
            border: 2px solid var(--color-border);
            resize: vertical;
        }
        
        /* New Dive UI Styles */
        #dive-ui-container, #deep-dive-ui-container {
            display: flex;
            gap: 2rem;
            align-items: center;
            justify-content: space-between;
            width: 100%;
            max-width: 800px;
            margin-top: 1rem;
        }
        
        #dive-stats, #deep-dive-stats {
            display: flex;
            gap: 1rem;
            align-items: center;
        }
        
        #dive-inventory-display, #deep-dive-inventory-display {
            border: 2px solid var(--color-border);
            padding: 0.5rem;
            min-height: 60px;
            width: 300px;
            text-align: left;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 0.5rem;
        }

        /* Landmark Modal Choices */
        #landmark-modal-container {
            display: none;
            flex-direction: row;
            gap: 1rem;
            align-items: flex-start;
        }

        #landmark-modal-container .modal-content {
            width: 100%;
        }

        #dive-inventory-popup {
            border: 2px solid var(--color-border);
            padding: 1rem;
            background-color: var(--color-bg);
            width: 300px;
        }
        
        #landmark-choices {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
            margin: 1rem 0;
        }
        
        #landmark-actions {
            display: flex;
            justify-content: center;
            gap: 1rem;
        }
        
        /* Statistics Screen Styles */
        .statistics-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            width: 100%;
            height: 100%;
            padding: 2rem;
            overflow-y: auto;
        }
        
        .stats-container {
            width: 100%;
            max-width: 600px;
            text-align: left;
        }
        
        .stats-container h2 {
            border-bottom: 2px solid var(--color-border);
            padding-bottom: 0.5rem;
            margin-bottom: 1rem;
        }

        /* Speedrun Timer Styles */
        .speedrun-timer {
            position: fixed;
            bottom: 50px; /* Raised up */
            left: 20px;
            font-family: 'Courier New', Courier, monospace;
            font-size: 2rem;
            font-weight: bold;
            color: var(--color-fg);
            background-color: var(--color-bg);
            border: 2px solid var(--color-border);
            padding: 0.5rem 1rem;
            z-index: 5;
        }
        
        /* Owner Credit Styles */
        .owner-credit {
            position: fixed;
            bottom: 20px;
            left: 20px;
            z-index: 5;
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 1rem;
            font-weight: bold;
        }
        
        .owner-credit img {
            height: 24px;
            width: 24px;
        }
        
        /* Wave Animation */
        .ocean {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 20%;
            z-index: -1;
            overflow: hidden;
        }

        .wave {
            background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 800 88.7'%3E%3Cpath d='M800 56.9c-155.5 0-204.9-50-405.5-49.9-200 0-250 49.9-394.5 49.9v31.8h800v-.2-31.6z' fill='%23000000'/%3E%3C/svg%3E");
            position: absolute;
            width: 200%;
            height: 100%;
            animation: wave 10s -3s linear infinite;
            transform: translate3d(0, 0, 0);
            opacity: 0.1;
        }

        .wave:nth-of-type(2) {
            bottom: 0;
            animation: wave 18s linear reverse infinite;
            opacity: 0.1;
        }

        .wave:nth-of-type(3) {
            bottom: 0;
            animation: wave 20s -1s linear infinite;
            opacity: 0.05;
        }

        @keyframes wave {
            0% {transform: translateX(0);}
            50% {transform: translateX(-25%);}
            100% {transform: translateX(-50%);}
        }
        
        /* Bestiary Styles */
        .bestiary-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            width: 100%;
            height: 100%;
            padding: 2rem;
            overflow-y: auto;
        }
        
        .bestiary-entry {
            border: 2px solid var(--color-border);
            padding: 1rem;
            margin-bottom: 1rem;
            width: 100%;
            max-width: 600px;
        }
        
        .bestiary-entry h3 {
            margin-top: 0;
        }
        
        /* Nursery and Merchant Styles */
        .nursery-page, .merchant-page {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            width: 100%;
            height: 100%;
            padding: 2rem;
            overflow-y: auto;
        }
        
        #incubator-slots {
            display: flex;
            gap: 1rem;
            margin-bottom: 2rem;
        }

        .incubator-slot {
            width: 100px;
            height: 100px;
            border: 2px solid var(--color-border);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
        }
        
        .incubator-slot.shaking {
            animation: shake 0.5s infinite;
        }

        @keyframes shake {
            0%, 100% { transform: translate(0, 0) rotate(0); }
            25% { transform: translate(2px, -2px) rotate(2deg); }
            50% { transform: translate(-2px, 2px) rotate(-2deg); }
            75% { transform: translate(2px, 2px) rotate(2deg); }
        }
        
        #pet-catalog-container {
            width: 100%;
            max-width: 800px;
        }
        
        .pet-rarity-section {
            margin-bottom: 1rem;
        }
        
        .pet-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(60px, 1fr));
            gap: 5px;
        }

        .pet-slot {
            width: 60px;
            height: 60px;
            border: 2px solid var(--color-border);
            background-color: #eee;
            filter: blur(4px);
            position: relative;
            cursor: pointer;
        }
        
        .pet-slot.unlocked {
            filter: none;
            background-color: var(--color-bg);
        }
        
        .pet-slot.equipped {
            border-color: blue;
            border-width: 3px;
        }

        .pet-tooltip {
            visibility: hidden;
            width: 200px;
            background-color: var(--color-bg);
            border: 2px solid var(--color-border);
            color: var(--color-fg);
            text-align: left;
            padding: 1rem;
            position: absolute;
            z-index: 1;
            bottom: 105%;
            left: 50%;
            margin-left: -100px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .pet-slot:hover .pet-tooltip {
            visibility: visible;
            opacity: 1;
        }
        
        #merchant-eggs {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 1rem;
            margin-top: 2rem;
        }

        .egg-for-sale {
            width: 200px;
            height: 200px;
            border: 2px solid var(--color-border);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            padding: 1rem;
            cursor: pointer;
        }
        
        .egg-for-sale:hover {
            background-color: var(--color-button-hover);
        }
        
        .egg-art {
            font-size: 3rem;
        }
        
        #pearl-display {
            border: 2px solid var(--color-border);
            padding: 0.5rem 1rem;
            font-size: 1.2rem;
        }
        
        #hatch-modal .modal-content {
            overflow: hidden;
        }

        #wheel-container {
            width: 300px;
            height: 300px;
            border: 2px solid var(--color-border);
            border-radius: 50%;
            position: relative;
            margin: 1rem auto;
            transition: transform 5s cubic-bezier(0.25, 1, 0.5, 1);
        }

        .wheel-segment {
            position: absolute;
            width: 100%;
            height: 100%;
            clip-path: polygon(50% 50%, 0% 0%, 100% 0%);
            transform-origin: 50% 50%;
        }
        
        #wheel-arrow {
            position: absolute;
            top: -20px;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 10px solid transparent;
            border-right: 10px solid transparent;
            border-bottom: 20px solid var(--color-fg);
        }


    </style>
</head>
<body>
    <div class="game-container-wrapper">
        <div class="game-container">
            <!-- Navigation Tabs -->
            <div class="navigation-tabs" style="display: none;">
                <button id="game-tab" class="tab-button active">Game</button>
                <button id="resource-tab" class="tab-button">Resource Gathering</button>
                <button id="crafting-tab" class="tab-button">Crafting Area<span id="crafting-indicator" class="new-indicator" style="display: none;"></span></button>
                <button id="achievements-tab" class="tab-button">Achievements</button>
                <button id="workers-tab" class="tab-button" style="display: none;">Workers<span id="workers-indicator" class="new-indicator" style="display: none;"></span></button>
                <button id="dive-tab" class="tab-button" style="display: none;">Dive<span id="dive-indicator" class="new-indicator" style="display: none;"></span></button>
                <button id="deep-dive-tab" class="tab-button" style="display: none;">Deep Dive</button>
                <button id="nursery-tab" class="tab-button" style="display: none;">Nursery</button>
                <button id="merchant-tab" class="tab-button" style="display: none;">Merchant</button>
            </div>
            
            <!-- Main Game Area -->
            <div id="game-screen" class="game-main-area" style="display: flex;">
                <div class="narrative-panel">
                    <div id="narrative-lines-container">
                        <p class="narrative-line">You wake up to the rhythmic lapping of cold water against a creaking hull. A flimsy boat, lost on an endless, black ocean. Your hands are tied with a thick, coarse rope. All you can do is search your surroundings for a way to break free.</p>
                    </div>
                </div>
                <!-- The main interactive button, now positioned at the bottom -->
                <div class="main-button-container">
                    <button id="main-button" class="game-button">
                        <span id="button-text">Search</span>
                        <div id="cooldown-bar" class="cooldown-bar" style="width: 0;"></div>
                    </button>
                </div>
                <!-- Speedrun Timer -->
                 <div id="speedrun-timer" class="speedrun-timer">00:00:00</div>
                 <!-- Owner Credit -->
                 <div class="owner-credit">
                     <span>ALoyfulGuy</span>
                     <img src="https://upload.wikimedia.org/wikipedia/commons/9/91/Octicons-mark-github.svg" alt="GitHub">
                     <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/97/Netlify_logo_%282%29.svg/1200px-Netlify_logo_%282%29.svg.png" alt="Netlify">
                 </div>
            </div>

            <!-- Resource Gathering Area Page -->
            <div id="resource-gathering-screen" class="resource-gathering-page">
                <p>RESOURCE GATHERING</p>
                <button id="gather-wood-button" class="game-button">
                    <span id="gather-wood-button-text">Gather Wood</span>
                    <div id="cooldown-bar-gather" class="cooldown-bar" style="width: 0;"></div>
                </button>
                <button id="swim-button" class="game-button" style="display: none;">
                    <span id="swim-button-text">Swim</span>
                    <div id="cooldown-bar-swim" class="cooldown-bar" style="width: 0;"></div>
                </button>
            </div>
            
            <!-- Dive Area Page -->
            <div id="dive-screen" class="dive-page">
                <div id="dive-prep-area">
                    <h2>Prepare for your Dive</h2>
                    <p>Select the tools you wish to bring with you.</p>
                    <div id="tool-selection-container" class="tool-selection-container">
                        <!-- Tools will be dynamically added here -->
                    </div>
                    <button id="begin-dive-button" class="game-button">
                        Begin Dive
                        <div id="cooldown-bar-dive" class="cooldown-bar" style="width: 0;"></div>
                    </button>
                </div>
                <div id="dive-minigame-area">
                    <div id="minigame-grid"></div>
                    <div id="dive-ui-container">
                        <div id="dive-stats">
                            <p>Moves Left: <span id="moves-left">20</span></p>
                            <div>
                                <p>Health: <span id="dive-health-text">15 / 15</span></p>
                                <div class="health-bar-container" style="height: 8px; width: 100px;">
                                    <div id="dive-player-health-bar" class="health-bar"></div>
                                </div>
                            </div>
                            <button id="use-bandage-button" class="game-button">Use Bandage (<span id="dive-bandage-count">0</span>)</button>
                             <button id="build-habitat-button" class="game-button" style="display: none;">Build Habitat</button>
                        </div>
                        <div id="dive-inventory-display">
                            <!-- Dive inventory slots will be shown here -->
                        </div>
                    </div>
                    <p>Use arrow keys or WASD to move. Find # for loot. Return to $ to finish.</p>
                </div>
            </div>

            <!-- Deep Dive Area Page -->
            <div id="deep-dive-screen" class="dive-page">
                <div id="deep-dive-prep-area">
                    <h2>Prepare for your Deep Dive</h2>
                    <p>The crushing depths await. Select your tools carefully.</p>
                    <div id="deep-tool-selection-container" class="tool-selection-container">
                        <!-- Tools will be dynamically added here -->
                    </div>
                    <button id="begin-deep-dive-button" class="game-button">
                        Begin Deep Dive
                        <div id="cooldown-bar-deep-dive" class="cooldown-bar" style="width: 0;"></div>
                    </button>
                </div>
                <div id="deep-dive-minigame-area">
                    <div id="deep-minigame-grid"></div>
                    <div id="deep-dive-ui-container">
                        <div id="deep-dive-stats">
                            <p>Moves Left: <span id="deep-moves-left">5</span></p>
                            <div>
                                <p>Health: <span id="deep-dive-health-text">15 / 15</span></p>
                                <div class="health-bar-container" style="height: 8px; width: 100px;">
                                    <div id="deep-dive-player-health-bar" class="health-bar"></div>
                                </div>
                            </div>
                            <button id="deep-use-bandage-button" class="game-button">Use Bandage (<span id="deep-dive-bandage-count">0</span>)</button>
                        </div>
                        <div id="deep-dive-inventory-display">
                            <!-- Deep Dive inventory slots will be shown here -->
                        </div>
                    </div>
                    <p>Arrow keys or WASD to move. The abyss is hostile. Return to $ to finish.</p>
                </div>
            </div>
            
            <!-- Crafting Area Page -->
            <div id="crafting-screen" class="craft-area-page">
                <h2>CRAFTING</h2>
                <div id="crafting-recipes-container" class="crafting-recipes-container">
                    <div class="crafting-recipe" id="woodpile-recipe">
                        <p>Unlit Woodpile</p>
                        <p>Requires: 5 Wood</p>
                        <button id="craft-woodpile-button" class="game-button">Craft</button>
                    </div>
                </div>
                <div id="purity-crafting-container" style="display: none; margin-top: 2rem; width: 100%;">
                    <h2>Purity Crafting</h2>
                    <div id="purity-recipes-container" class="crafting-recipes-container">
                        <!-- Purity recipes will be added here -->
                    </div>
                </div>
                 <div id="deep-sea-crafting-container" style="display: none; margin-top: 2rem; width: 100%;">
                    <h2>Deep Sea Crafting</h2>
                    <div id="deep-sea-recipes-container" class="crafting-recipes-container">
                        <!-- Deep Sea recipes will be added here -->
                    </div>
                </div>
            </div>
            
            <!-- New Workers Page -->
            <div id="workers-screen" class="workers-page">
                <h1>WORKERS</h1>
                <div class="worker-stats">
                    Total Workers: <span id="total-workers-count">0</span>
                    <br>
                    Wood Gatherers: <span id="wood-gatherers-count">0</span>
                    <br>
                    Swimmers: <span id="swimmers-count">0</span>
                    <br>
                    Idle Workers: <span id="idle-workers-count">0</span>
                </div>
                <div class="worker-assignment-container">
                    <div class="assignment-option">
                        <p>Assign a worker to gather wood.</p>
                        <button id="assign-wood-gatherer" class="assign-button">Assign</button>
                    </div>
                    <div class="assignment-option">
                        <p>Assign a worker to swim for materials.</p>
                        <button id="assign-swimmer" class="assign-button">Assign</button>
                    </div>
                    <div class="assignment-option">
                        <p>Recall a wood gatherer.</p>
                        <button id="recall-wood-gatherer" class="assign-button">Recall</button>
                    </div>
                    <div class="assignment-option">
                        <p>Recall a swimmer.</p>
                        <button id="recall-swimmer" class="assign-button">Recall</button>
                    </div>
                </div>
            </div>
            
            <!-- Nursery Page -->
            <div id="nursery-screen" class="nursery-page">
                <h1>Nursery</h1>
                <h3>Incubators</h3>
                <div id="incubator-slots"></div>
                <div id="egg-inventory-container" style="margin-bottom: 2rem; width: 100%; max-width: 600px;">
                    <h3>Your Eggs</h3>
                    <div id="egg-inventory-display" style="border: 2px solid var(--color-border); min-height: 80px; width: 100%; padding: 0.5rem; display: flex; gap: 0.5rem; flex-wrap: wrap;"></div>
                </div>
                <div id="pet-catalog-container">
                    <h3>Pet Catalog</h3>
                    <!-- Pet sections will be generated here -->
                </div>
            </div>

            <!-- Merchant Page -->
            <div id="merchant-screen" class="merchant-page">
                <h1>Egg Merchant</h1>
                <div id="pearl-display">Pearls: 0</div>
                <div id="merchant-eggs"></div>
            </div>

            
            <!-- Achievements Area Page -->
            <div id="achievements-screen" class="achievement-area-page">
                <h1>Achievements</h1>
                <div id="achievement-list-container" style="width: 100%; max-width: 600px;">
                    <div class="achievement-rarity-section"><h2>Common</h2></div>
                    <div class="achievement-rarity-section"><h2>Rare</h2></div>
                    <div class="achievement-rarity-section"><h2>Legendary</h2></div>
                    <div class="achievement-rarity-section"><h2>Mythical</h2></div>
                    <div class="achievement-rarity-section"><h2>Secret</h2></div>
                </div>
                <button id="back-button" class="game-button back-button">Back</button>
            </div>
            
            <!-- Progression Area Page -->
            <div id="progression-screen" class="progression-page">
                <h1>Story Progression</h1>
                <div class="progression-graph">
                    <div class="progression-section">Section 1: Regain Composure</div>
                    <div class="connector-line"></div>
                    <div id="section-2" class="progression-section locked">Section 2: Beginning to Begin</div>
                    <div class="connector-line"></div>
                    <div id="section-3" class="progression-section locked">Section 3: Deep-Sea Dweller</div>
                    <div class="connector-line"></div>
                    <div id="section-4" class="progression-section locked">Section 4: The Beacon</div>
                    <div class="connector-line"></div>
                    <div id="section-5" class="progression-section locked">Section 5: The Rescue</div>
                </div>
                <button id="progression-back-button" class="game-button back-button">Back</button>
            </div>
            
            <!-- Commands Area Page -->
            <div id="commands-screen" class="commands-page">
                <h1>Admin Commands</h1>
                <p>Enter an access code.</p>
                <input type="text" id="admin-code-input" class="admin-input" placeholder="Code...">
                <button id="admin-submit-button" class="game-button">Submit</button>
                <p id="admin-status"></p>
                <div id="admin-controls-area">
                    <div class="toggle-switch" id="free-crafting-toggle-container">
                        <span>Free Crafting</span>
                        <input type="checkbox" id="free-crafting-toggle" /><label for="free-crafting-toggle">Toggle</label>
                    </div>
                    <div class="toggle-switch" id="fast-cooldowns-toggle-container">
                        <span>Fast Cooldowns</span>
                        <input type="checkbox" id="fast-cooldowns-toggle" /><label for="fast-cooldowns-toggle">Toggle</label>
                    </div>
                    <div class="toggle-switch" id="infinite-moves-toggle-container">
                        <span>Infinite Dive Moves</span>
                        <input type="checkbox" id="infinite-moves-toggle" /><label for="infinite-moves-toggle">Toggle</label>
                    </div>
                    <div class="toggle-switch" id="god-health-toggle-container">
                        <span>500 Dive Health</span>
                        <input type="checkbox" id="god-health-toggle" /><label for="god-health-toggle">Toggle</label>
                    </div>
                    <div class="toggle-switch" id="no-enemies-toggle-container">
                        <span>No Enemies</span>
                        <input type="checkbox" id="no-enemies-toggle" /><label for="no-enemies-toggle">Toggle</label>
                    </div>
                </div>
                <button id="commands-back-button" class="game-button back-button">Back</button>
            </div>
            
            <!-- Save/Load Page -->
            <div id="save-load-screen" class="save-load-page">
                <h1>Save & Load Game</h1>
                <div class="save-load-section">
                    <h3>Export</h3>
                    <p>Copy this text and save it somewhere safe.</p>
                    <textarea id="export-textarea" class="save-load-textarea" readonly></textarea>
                </div>
                <div class="save-load-section">
                    <h3>Import</h3>
                    <p>Paste your saved game data here.</p>
                    <textarea id="import-textarea" class="save-load-textarea"></textarea>
                    <button id="import-button" class="game-button">Import</button>
                </div>
                <button id="save-load-back-button" class="game-button back-button">Back</button>
            </div>

            <!-- Statistics Page -->
            <div id="statistics-screen" class="statistics-page">
                <h1>Statistics</h1>
                <div id="stats-container" class="stats-container">
                    <!-- Stats will be dynamically added here -->
                </div>
                <button id="bestiary-button" class="game-button" style="margin-top: 1rem;">Bestiary</button>
                <button id="statistics-back-button" class="game-button back-button">Back</button>
            </div>
            
            <!-- Bestiary Page -->
            <div id="bestiary-screen" class="bestiary-page">
                <h1>Bestiary</h1>
                <div id="bestiary-container" class="stats-container">
                    <!-- Enemy info will be dynamically added here -->
                </div>
                <button id="bestiary-back-button" class="game-button back-button">Back to Stats</button>
            </div>


            <!-- Inventory Panel (initially hidden) -->
            <div id="inventory-panel" class="inventory-panel">
                <div class="inventory-box">
                    <p>INVENTORY</p>
                    <div id="knife-item" class="inventory-item clickable" style="display: none;">
                        <span>Rusted Knife</span>
                    </div>
                    <div id="axe-item" class="inventory-item disabled" style="display: none;">
                        <span>Axehead</span>
                    </div>
                    <div id="wood-item" class="inventory-item" style="display: none;">
                        <span>Wood: </span>
                        <span id="wood-count">0</span>
                    </div>
                     <div id="screws-item" class="inventory-item" style="display: none;">
                        <span>Screws: </span>
                        <span id="screws-count">0</span>
                    </div>
                    <div id="unlit-woodpile-item" class="inventory-item clickable" style="display: none;">
                        <span>Unlit Woodpile</span>
                    </div>
                    <div id="flammable-liquid-item" class="inventory-item" style="display: none;">
                        <span>Flammable Liquid</span>
                    </div>
                    <div id="match-item" class="inventory-item" style="display: none;">
                        <span>Single Match</span>
                    </div>
                    <!-- New inventory items for the swim game -->
                    <div id="stone-item" class="inventory-item" style="display: none;">
                        <span>Stone: </span>
                        <span id="stone-count">0</span>
                    </div>
                    <div id="fabric-item" class="inventory-item" style="display: none;">
                        <span>Fabric: </span>
                        <span id="fabric-count">0</span>
                    </div>
                    <div id="rope-item" class="inventory-item" style="display: none;">
                        <span>Rope: </span>
                        <span id="rope-count">0</span>
                    </div>
                    <div id="metallic-scrap-item" class="inventory-item" style="display: none;">
                        <span>Metallic Scrap: </span>
                        <span id="metallic-scrap-count">0</span>
                    </div>
                    <!-- Crafted items -->
                    <div id="diving-gear-item" class="inventory-item" style="display: none;">
                        <span>Simple Diving Gear</span>
                    </div>
                     <div id="dive-bag-item" class="inventory-item" style="display: none;">
                        <span>Dive Bag</span>
                    </div>
                     <div id="furnace-item" class="inventory-item" style="display: none;">
                        <span>Furnace</span>
                    </div>
                    <div id="spear-item" class="inventory-item" style="display: none;">
                        <span>Spear</span>
                    </div>
                    <div id="air-tank-item" class="inventory-item" style="display: none;">
                        <span>Better Air Tank</span>
                    </div>
                     <!-- New Dive Items -->
                    <div id="pearl-item" class="inventory-item" style="display: none;">
                        <span>Pearl: </span>
                        <span id="pearl-count">0</span>
                    </div>
                    <div id="harpoon-gun-item" class="inventory-item" style="display: none;">
                        <span>Harpoon Gun</span>
                    </div>
                    <div id="harpoon-ammo-item" class="inventory-item" style="display: none;">
                        <span>Harpoon Ammo: </span>
                        <span id="harpoon-ammo-count">0</span>
                    </div>
                    <div id="fishing-net-item" class="inventory-item" style="display: none;">
                        <span>Fishing Net</span>
                    </div>
                    <div id="bandage-item" class="inventory-item" style="display: none;">
                        <span>Bandage: </span>
                        <span id="bandage-count">0</span>
                    </div>
                    <!-- Purity Items -->
                    <div id="tungsten-item" class="inventory-item" style="display: none;">
                        <span>Tungsten: </span>
                        <span id="tungsten-count">0</span>
                    </div>
                    <div id="kevlar-item" class="inventory-item" style="display: none;">
                        <span>Kevlar: </span>
                        <span id="kevlar-count">0</span>
                    </div>
                     <div id="kevlar-armour-item" class="inventory-item" style="display: none;">
                        <span>Kevlar Armour</span>
                    </div>
                    <div id="reinforced-stone-item" class="inventory-item" style="display: none;">
                        <span>Reinforced Stone: </span>
                        <span id="reinforced-stone-count">0</span>
                    </div>
                    <div id="reinforced-wood-item" class="inventory-item" style="display: none;">
                        <span>Reinforced Wood: </span>
                        <span id="reinforced-wood-count">0</span>
                    </div>
                    <!-- Deep Dive Items -->
                    <div id="complex-diving-gear-item" class="inventory-item" style="display: none;">
                        <span>Complex Diving Gear</span>
                    </div>
                    <div id="coral-fragments-item" class="inventory-item" style="display: none;"><span>Coral Fragments: </span><span id="coral-fragments-count">0</span></div>
                    <div id="hydrothermal-sludge-item" class="inventory-item" style="display: none;"><span>Hydrothermal Sludge: </span><span id="hydrothermal-sludge-count">0</span></div>
                    <div id="bioluminescence-sac-item" class="inventory-item" style="display: none;"><span>Bio-Luminescence Sac: </span><span id="bioluminescence-sac-count">0</span></div>
                    <div id="electroluminescent-filament-item" class="inventory-item" style="display: none;"><span>Electroluminescent Filament: </span><span id="electroluminescent-filament-count">0</span></div>
                    <div id="chitinous-plating-item" class="inventory-item" style="display: none;"><span>Chitinous Plating: </span><span id="chitinous-plating-count">0</span></div>
                    <div id="nautilus-shell-item" class="inventory-item" style="display: none;"><span>Nautilus Shell: </span><span id="nautilus-shell-count">0</span></div>
                    <div id="ethereal-viscera-item" class="inventory-item" style="display: none;"><span>Ethereal Viscera: </span><span id="ethereal-viscera-count">0</span></div>
                    <div id="hydrothermal-vent-mineral-item" class="inventory-item" style="display: none;"><span>Hydrothermal Vent Mineral: </span><span id="hydrothermal-vent-mineral-count">0</span></div>
                    <div id="black-ice-core-item" class="inventory-item" style="display: none;"><span>Black Ice Core: </span><span id="black-ice-core-count">0</span></div>
                    <div id="giant-squid-beak-item" class="inventory-item" style="display: none;"><span>Giant Squid Beak: </span><span id="giant-squid-beak-count">0</span></div>
                    <div id="abyssal-pearl-item" class="inventory-item" style="display: none;"><span>Abyssal Pearl: </span><span id="abyssal-pearl-count">0</span></div>
                    <div id="mythic-tooth-item" class="inventory-item" style="display: none;"><span>Mythic Tooth: </span><span id="mythic-tooth-count">0</span></div>
                    <div id="bone-fragment-item" class="inventory-item" style="display: none;"><span>Bone Fragment: </span><span id="bone-fragment-count">0</span></div>
                </div>
            </div>
        </div>
    </div>

    <!-- The modal popups -->
    <div id="cut-rope-modal" class="modal">
        <div class="modal-content">
            <p>You have found a rusted knife. Use it to cut the ropes?</p>
            <button id="cut-rope-button" class="modal-button">Cut the Rope</button>
        </div>
    </div>
    
    <div id="light-fire-modal" class="modal">
        <div class="modal-content">
            <p>You have a woodpile and a match. Use them to light the fire?</p>
            <button id="light-fire-button" class="modal-button">Light the Pile</button>
        </div>
    </div>
    
    <!-- New modal for welcoming a new crew member -->
    <div id="new-person-modal" class="modal">
        <div class="modal-content">
            <p>A small, stray raft with a person on it has gently collided with your boat. What will you do?</p>
            <button id="take-in-person-button" class="modal-button">Take In</button>
        </div>
    </div>
    
    <!-- New modal for reinforced shelter crew members -->
    <div id="new-people-modal" class="modal">
        <div class="modal-content">
            <p>2 people on 1 raft gently collided with your boat, they seem relieved to see human faces.</p>
            <button id="take-in-people-button" class="modal-button">Take Them In</button>
        </div>
    </div>

    
    <!-- New modal for import confirmation -->
    <div id="import-confirm-modal" class="modal">
        <div class="modal-content">
            <p>Are you sure you want to import this save? Your current progress will be lost.</p>
            <button id="confirm-import-button" class="modal-button">Yes, Import</button>
            <button id="cancel-import-button" class="modal-button">Cancel</button>
        </div>
    </div>
    
    <!-- Hatching Modal -->
    <div id="hatch-modal" class="modal">
        <div class="modal-content">
            <h3>Hatching Pet...</h3>
            <div id="wheel-arrow"></div>
            <div id="wheel-container"></div>
            <button id="hatch-result-button" class="modal-button" style="display:none;">Confirm</button>
        </div>
    </div>

    
    <!-- The music control button and audio tag -->
    <button id="music-control" class="music-control-button">Music: Off</button>
    <button id="achievements-button" class="music-control-button" style="top: 70px;">Achievements</button>
    <button id="progression-button" class="music-control-button" style="top: 120px;">Progression</button>
    <button id="commands-button" class="music-control-button" style="top: 170px;">Commands</button>
    <button id="save-button" class="music-control-button" style="top: 220px;">Save</button>
    <button id="statistics-button" class="music-control-button" style="top: 270px;">Statistics</button>
    <audio id="background-music" loop src="https://cdn.pixabay.com/download/audio/2022/08/04/audio_2dde649a37.mp3"></audio>
    <audio id="click-sound" src="https://cdn.pixabay.com/download/audio/2021/08/04/audio_c6ccf3233c.mp3"></audio>


    <!-- Achievement pop-up container -->
    <div id="achievement-container" class="achievement-container"></div>
    
    <!-- New screen for the swimming loading animation -->
    <div id="swim-loading-screen" class="swim-loading-screen">
        <div class="bouncing-text">
            <span>S</span><span>w</span><span>i</span><span>m</span><span>m</span><span>i</span><span>n</span><span>g</span>
            <span> </span>
            <span>F</span><span>o</span><span>r</span>
            <span> </span>
            <span>S</span><span>o</span><span>m</span><span>e</span><span>t</span><span>h</span><span>i</span><span>n</span><span>g</span><span>.</span><span>.</span><span>.</span>
        </div>
        <div class="loading-bar-container">
            <div id="loading-bar" class="loading-bar"></div>
        </div>
    </div>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Deep Sea - Nursery & Merchant</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: monospace; /* Changed font */
            background-color: #ffffff; /* White background */
            color: #111111; /* Black text */
            overflow: hidden;
        }
        .game-container {
            border: 4px solid #000000; /* Black border */
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.2); /* Softer shadow */
        }
        .game-screen {
            background-color: #ffffff; /* White background */
            border: 2px solid #cccccc; /* Light gray border */
        }
        .tab-button {
            border: 2px solid #cccccc;
            transition: all 0.3s ease;
        }
        .tab-button.active, .tab-button:hover {
            background-color: #000000; /* Black background on hover/active */
            color: #ffffff; /* White text on hover/active */
            border-color: #000000;
        }
        .box {
            border: 2px solid #cccccc;
        }
        .egg-box {
            cursor: pointer;
            transition: all 0.3s ease;
        }
        .egg-box:hover {
            border-color: #000000;
            transform: translateY(-5px);
        }
        .egg-symbol {
            font-size: 3rem;
            color: #000000;
        }
        .pet-square {
            border: 2px solid #cccccc;
            width: 60px;
            height: 60px;
            transition: all 0.3s ease;
        }
        .pet-square.blurred {
            filter: blur(4px);
            background-color: #eeeeee;
        }
        .pet-square:not(.blurred):hover {
            border-color: #000000;
            transform: scale(1.1);
            z-index: 10;
        }
        .pet-tooltip {
            display: none;
            position: absolute;
            background-color: #ffffff;
            border: 2px solid #000000;
            padding: 10px;
            z-index: 20;
            width: 250px;
            bottom: 100%;
            left: 50%;
            transform: translateX(-50%);
            margin-bottom: 5px;
        }
        .pet-square:hover .pet-tooltip {
            display: block;
        }
        .action-button {
            background-color: #f0f0f0;
            border: 2px solid #000000;
            color: #000000;
            transition: all 0.3s ease;
        }
        .action-button:hover {
            background-color: #000000;
            color: #ffffff;
        }
        .shaking {
            animation: shake 0.82s cubic-bezier(.36,.07,.19,.97) both infinite;
            transform: translate3d(0, 0, 0);
        }
        @keyframes shake {
            10%, 90% { transform: translate3d(-1px, 0, 0); }
            20%, 80% { transform: translate3d(2px, 0, 0); }
            30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
            40%, 60% { transform: translate3d(4px, 0, 0); }
        }
        .modal {
            background-color: rgba(255, 255, 255, 0.85);
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen">

    <div id="game-container" class="game-container w-full max-w-6xl h-[90vh] p-4 flex flex-col">
        <header class="flex justify-between items-center mb-4">
            <h1 class="text-4xl text-black">Nursery & Merchant</h1>
            <div id="pearl-display" class="box text-2xl p-2">Pearls: 0</div>
        </header>

        <div class="flex space-x-4 mb-4">
            <button id="nursery-tab-btn" class="tab-button p-2 text-xl flex-1 active">Nursery</button>
            <button id="merchant-tab-btn" class="tab-button p-2 text-xl flex-1">Merchant</button>
        </div>

        <main class="flex-1 overflow-hidden">
            <!-- Nursery Screen -->
            <div id="nursery-screen" class="game-screen p-4 h-full overflow-y-auto">
                <h2 class="text-2xl text-center text-black mb-4">Creature Nursery</h2>
                <div class="mb-6">
                    <h3 class="text-xl mb-2">Incubators</h3>
                    <div id="incubator-slots" class="grid grid-cols-5 gap-4"></div>
                </div>
                <div class="mb-6">
                    <h3 class="text-xl mb-2">Egg Inventory (<span id="egg-inventory-count">0</span>)</h3>
                    <div id="egg-inventory" class="flex flex-wrap gap-2 p-2 box min-h-[50px]"></div>
                </div>
                <div>
                    <h3 class="text-xl mb-2">Pet Catalog</h3>
                    <div id="pet-catalog" class="space-y-4"></div>
                </div>
            </div>

            <!-- Merchant Screen -->
            <div id="merchant-screen" class="game-screen p-4 h-full hidden">
                <h2 class="text-2xl text-center text-black mb-4">Pearl Merchant</h2>
                <div id="merchant-grid" class="grid grid-cols-3 gap-8 justify-items-center"></div>
            </div>
        </main>
    </div>
    
    <!-- Modal for Popups -->
    <div id="modal" class="modal fixed inset-0 items-center justify-center hidden">
        <div id="modal-content" class="box p-8 bg-white text-center max-w-lg w-full"></div>
    </div>

    <script>
        // --- CONSTANTS AND DATA ---
        const PETS = {
            'Nudibranch': { rarity: 'Common', perk: "Adaptive Defense - 5% chance to dodge an enemy's special ability." },
            'Seahorse': { rarity: 'Common', perk: "Swift Current - Increases dive moves by 5." },
            'Starfish': { rarity: 'Common', perk: "Regenerative Aid - Heals 1 health every 5 seconds." },
            'Sea Slug': { rarity: 'Common', perk: "Slime Trail - Enemies that hit you have their attack cooldowns slowed by 10% for 2s." },
            'Hermit Crab': { rarity: 'Common', perk: "Fortify - Increases max health by 10%." },
            'Anemone Shrimp': { rarity: 'Common', perk: "Symbiotic Guard - Places a small shield that absorbs the next 3 damage. (8s CD)" },
            'Spiny Water Flea': { rarity: 'Common', perk: "Piercing Swarm - Adds 1 extra damage to your attacks." },
            'Brittle Star': { rarity: 'Common', perk: "Cooldown Link - Decreases the other equipped pet's cooldown by 10%." },
            'Feather Star': { rarity: 'Common', perk: "Stealthy Current - Decreases enemies' chance to hit you by 10%." },
            'Sea Urchin': { rarity: 'Common', perk: "Spiky Aura - Deals 1 damage to enemies every time they attack you." },
            'Dumbo Octopus': { rarity: 'Uncommon', perk: "Deep Descent - Temporarily increases max health by 10% for 15s. (9s CD)" },
            'Cuttlefish': { rarity: 'Uncommon', perk: "Invisibility Ink - Grants 2s of invisibility, making enemies miss all attacks. (9s CD)" },
            'Pufferfish': { rarity: 'Uncommon', perk: "Cooldown Link - Decreases the other equipped pet's cooldown by 20%." },
            'Giant Clam': { rarity: 'Uncommon', perk: "Protective Shell - Creates a temporary shield that absorbs up to 8 damage. (8s CD)" },
            'Sea Turtle': { rarity: 'Uncommon', perk: "Enduring Shell - Increases defense by 20% for 8s. (9s CD)" },
            'Leafy Seadragon': { rarity: 'Uncommon', perk: "Camouflage - Enemies have an extra 15% chance to miss their attack." },
            'Flying Fish': { rarity: 'Uncommon', perk: "Evasive Leap - Dodge the first 5 attacks from an enemy. (10s CD)" },
            'Atlantic Blue Tang': { rarity: 'Uncommon', perk: "Quick-Heal - Increases effectiveness of all healing items by 20%." },
            'Vampire Crab': { rarity: 'Uncommon', perk: "Sanguine Strike - Your attacks have a 10% chance to heal you." },
            'Blue Ringed Octopus': { rarity: 'Uncommon', perk: "Venomous Touch - Your critical hit has a 15% chance to apply poison (7 dmg over 3s)." },
            'Lionfish': { rarity: 'Rare', perk: "Venomous Spines - Your melee attacks cause poison (2 dmg/s for 5s)." },
            'Bobtail Squid': { rarity: 'Rare', perk: "Light Pulse - Blinds enemies for 4s, increasing their attack CD by 20%. (9s CD)" },
            'Box Jellyfish': { rarity: 'Rare', perk: "Venomous Tentacle - Your ranged attacks have a 5% chance to deal 20 bonus poison damage." },
            'Glass Squid': { rarity: 'Rare', perk: "Crystal Shroud - Become invulnerable for 1s. (9s CD)" },
            'Bobbit Worm': { rarity: 'Rare', perk: "Cooldown Link - Decreases the other equipped pet's cooldown by 15%." },
            'Sarcastic Fringehead': { rarity: 'Rare', perk: "Fierce Territory - When at or below 5 health, deal 2 damage every 2s." },
            'Warty Frogfish': { rarity: 'Rare', perk: "Angler's Bait - Lure an enemy, causing them to take 20% more damage for 5s. (9s CD)" },
            'Pink See-Through Fantasia': { rarity: 'Rare', perk: "Ethereal Flow - Reduces all damage taken from enemy attacks by 25%." },
            'Strawberry Squid': { rarity: 'Rare', perk: "Lightburst - Disorients an enemy for 2s, increasing their attack CD by 25%. (9s CD)" },
            'Flamingo Tongue Snail': { rarity: 'Rare', perk: "Deflecting Shell - Has a 10% chance to reflect an attack back at the enemy." },
            'Sea Angel': { rarity: 'Epic', perk: "Spirit's Grace - Heals you for 3 health per second for 5s. (9s CD)" },
            'Yeti Crab': { rarity: 'Epic', perk: "Frostbite - Your attacks have a 10% chance to freeze the enemy for 2s." },
            'Comb Jelly': { rarity: 'Epic', perk: "Bioluminescent Trail - Deals 2 damage per second and increases enemy attack CD by 20%." },
            'Vampire Squid': { rarity: 'Epic', perk: "Abyssal Shield - Creates a shield that absorbs up to 25 damage. (9s CD)" },
            'Barreleye Fish': { rarity: 'Epic', perk: "Clarity of Sight - Reveals enemy weak points, granting a 25% bonus damage multiplier." },
            'Coelacanth': { rarity: 'Epic', perk: "Cooldown Link - Decreases the other equipped pet's cooldown by 20%." },
            'Bobbit Worm Epic': { rarity: 'Epic', perk: "Tunneling Strike - Burrow for 2s, making you invulnerable. (9s CD)" },
            'Giant Spider Crab': { rarity: 'Epic', perk: "Pincer Crush - A single attack dealing 10 damage that can stun for 3s. (9s CD)" },
            'Flashlight Fish': { rarity: 'Epic', perk: "Blinding Flash - Stuns a single enemy for 2s. (9s CD)" },
            'Cookiecutter Shark': { rarity: 'Epic', perk: "Bite of the Abyss - Your attacks apply a bleed effect (2 dmg/s for 1s)." },
            'Giant Isopod': { rarity: 'Legendary', perk: "Hardened Exoskeleton - Grants a 50% increase to armor for 5s. (9s CD)" },
            'Giant Tube Worm': { rarity: 'Legendary', perk: "Ventilation Blast - Creates an explosion dealing 18 damage. (9s CD)" },
            'Snailfish': { rarity: 'Legendary', perk: "Evasive Slime - Increases your dodge chance by 30% for 3s. (9s CD)" },
            'Moon Jellyfish': { rarity: 'Legendary', perk: "Calming Aura - Heals you to 50 health and removes all negative effects. (9s CD)" },
            'Dragonfish': { rarity: 'Legendary', perk: "Cooldown Link - Decreases the other equipped pet's cooldown by 25%." },
            'Pacific Blackdragon': { rarity: 'Legendary', perk: "Shadow Meld - Become invisible and immune to damage for 3s. (9s CD)" },
            'Wobbegong Shark': { rarity: 'Legendary', perk: "Ambush Hunter - Grants a 40% damage bonus to attacks from behind." },
            'Manta Ray': { rarity: 'Legendary', perk: "Abyssal Glide - Gain 3s of complete invulnerability. (9s CD)" },
            'Gulper Eel': { rarity: 'Legendary', perk: "Lacerating Bite - Deals 10 damage. (9s CD)" },
            'Mimic Octopus': { rarity: 'Legendary', perk: "Adaptive Mimicry - Directs an enemy's special ability back at them with 50% less effect. (9s CD)" },
            "Lion's Mane Jellyfish": { rarity: 'Mythic', perk: "Venomous Barrage - Deals 1 dmg/s for 20s. (1 use per encounter)" },
            'Chimaera': { rarity: 'Mythic', perk: "Living Fossil - Grants a permanent 50% health boost and 15% player attack CD reduction." },
            'Immortal Jellyfish': { rarity: 'Secret', perk: "Rejuvenation Cycle - On death, revives you to 100% health and grants a 20% damage boost for 10s. (1 use per encounter)" }
        };
        
        const EGG_DATA = {
            'Common Egg': { price: 800, time: 5 * 60 * 1000, chances: { 'Common': 0.70, 'Uncommon': 0.20, 'Rare': 0.10 } },
            'Complex Egg': { price: 2400, time: 15 * 60 * 1000, chances: { 'Uncommon': 0.50, 'Rare': 0.30, 'Epic': 0.15, 'Common': 0.05 } },
            'Elite Egg': { price: 8000, time: 30 * 60 * 1000, chances: { 'Rare': 0.45, 'Epic': 0.40, 'Legendary': 0.10, 'Uncommon': 0.05 } },
            'Apocryphal Egg': { price: 20000, time: 60 * 60 * 1000, chances: { 'Legendary': 0.65, 'Mythical': 0.20, 'Epic': 0.15 } },
            'Secret Egg': { price: 32000, time: 2 * 60 * 60 * 1000, chances: { 'Legendary': 0.45, 'Mythical': 0.50, 'Secret': 0.05 } }
        };

        const RARITIES = ['Common', 'Uncommon', 'Rare', 'Epic', 'Legendary', 'Mythic', 'Secret'];

        // --- GAME STATE ---
        let gameState = {
            pearls: 0, // Starting pearls for testing
            eggs: [],
            incubators: [null], // Start with one incubator
            discoveredPets: {},
            equippedPets: [],
            currentScreen: 'nursery',
            selectedEggIndex: null
        };

        // --- DOM ELEMENTS ---
        const pearlDisplay = document.getElementById('pearl-display');
        const nurseryTabBtn = document.getElementById('nursery-tab-btn');
        const merchantTabBtn = document.getElementById('merchant-tab-btn');
        const screens = {
            nursery: document.getElementById('nursery-screen'),
            merchant: document.getElementById('merchant-screen')
        };
        const modal = document.getElementById('modal');
        const modalContent = document.getElementById('modal-content');

        // --- CORE GAME LOGIC ---
        function saveGame() {
            localStorage.setItem('deepSeaPetState', JSON.stringify(gameState));
        }

        function loadGame() {
            const savedState = localStorage.getItem('deepSeaPetState');
            if (savedState) {
                gameState = JSON.parse(savedState);
            }
        }
        
        function getPetIcon(petName) {
            const words = petName.replace(/[^a-zA-Z\s]/g, "").split(' ');
            if (words.length > 1) {
                return (words[0][0] + words[words.length - 1][0]).toUpperCase();
            }
            return (petName.substring(0, 2)).toUpperCase();
        }

        function switchTab(tabName) {
            gameState.currentScreen = tabName;
            Object.values(screens).forEach(s => s.classList.add('hidden'));
            screens[tabName].classList.remove('hidden');

            [nurseryTabBtn, merchantTabBtn].forEach(btn => btn.classList.remove('active'));
            document.getElementById(`${tabName}-tab-btn`).classList.add('active');
            updateUI();
        }

        function updateUI() {
            pearlDisplay.textContent = `Pearls: ${gameState.pearls.toLocaleString()}`;
            if (gameState.currentScreen === 'nursery') renderNursery();
            else if (gameState.currentScreen === 'merchant') renderMerchant();
            saveGame();
        }

        // --- MERCHANT LOGIC ---
        function renderMerchant() {
            const grid = document.getElementById('merchant-grid');
            grid.innerHTML = '';
            const eggLayout = ['Common Egg', 'Complex Egg', 'Elite Egg', null, 'Apocryphal Egg', 'Secret Egg'];

            eggLayout.forEach((eggName) => {
                if (!eggName) {
                    grid.appendChild(document.createElement('div'));
                    return;
                }
                const eggData = EGG_DATA[eggName];
                const box = document.createElement('div');
                box.className = 'egg-box box p-4 text-center w-64';
                box.innerHTML = `
                    <div class="text-2xl mb-2">${eggName}</div>
                    <div class="egg-symbol">⬯</div>
                    <div class="text-xl mt-2">${eggData.price.toLocaleString()} Pearls</div>
                `;
                box.onclick = () => buyEgg(eggName);
                grid.appendChild(box);
            });
        }

        function buyEgg(eggName) {
            const eggData = EGG_DATA[eggName];
            if (gameState.pearls >= eggData.price) {
                gameState.pearls -= eggData.price;
                gameState.eggs.push({ type: eggName, id: Date.now() });
                updateUI();
            } else {
                showMessage("Not enough pearls!");
            }
        }

        // --- NURSERY LOGIC ---
        function renderNursery() {
            renderIncubators();
            renderEggInventory();
            renderPetCatalog();
        }

        function renderIncubators() {
            const slots = document.getElementById('incubator-slots');
            slots.innerHTML = '';
            // Render existing slots
            gameState.incubators.forEach((egg, index) => {
                const slot = document.createElement('div');
                slot.className = 'box w-full h-32 flex items-center justify-center text-center p-2';
                if (egg) {
                    const timeRemaining = egg.hatchTime - Date.now();
                    if (timeRemaining <= 0) {
                        slot.innerHTML = `<div class="cursor-pointer shaking">
                            <div class="egg-symbol">⬯</div>
                            <div>Ready to Hatch!</div>
                        </div>`;
                        slot.onclick = () => hatchEgg(index);
                    } else {
                        slot.innerHTML = `<div>
                            <div class="egg-symbol">⬯</div>
                            <div>${egg.type}</div>
                            <div class="countdown">${new Date(timeRemaining).toISOString().substr(11, 8)}</div>
                        </div>`;
                    }
                } else {
                    slot.innerHTML = `<div class="text-4xl text-gray-400">+</div>`;
                    slot.onclick = () => placeEggInIncubator(index);
                }
                slots.appendChild(slot);
            });

            // Render the "buy new slot" button if applicable
            if (gameState.incubators.length < 5) {
                const cost = gameState.incubators.length * 100;
                const buySlot = document.createElement('div');
                buySlot.className = 'box w-full h-32 flex flex-col items-center justify-center text-center p-2 cursor-pointer hover:border-black transition-colors';
                buySlot.innerHTML = `
                    <div class="text-2xl text-black">+</div>
                    <div class="text-xl">Buy Slot</div>
                    <div class="text-lg">${cost.toLocaleString()} Pearls</div>
                `;
                buySlot.onclick = () => buyIncubatorSlot();
                slots.appendChild(buySlot);
            }
        }

        function buyIncubatorSlot() {
            if (gameState.incubators.length >= 5) return;
            const cost = gameState.incubators.length * 100;
            if (gameState.pearls >= cost) {
                gameState.pearls -= cost;
                gameState.incubators.push(null);
                updateUI();
            } else {
                showMessage("Not enough pearls to buy a new slot!");
            }
        }

        function renderEggInventory() {
            const inventory = document.getElementById('egg-inventory');
            inventory.innerHTML = '';
            document.getElementById('egg-inventory-count').textContent = gameState.eggs.length;
            gameState.eggs.forEach((egg, index) => {
                const eggDiv = document.createElement('div');
                eggDiv.className = 'box p-2 text-center cursor-pointer';
                if (gameState.selectedEggIndex === index) {
                    eggDiv.classList.add('border-black');
                }
                eggDiv.innerHTML = `<div class="text-2xl">⬯</div><div>${egg.type}</div>`;
                eggDiv.onclick = () => {
                    gameState.selectedEggIndex = (gameState.selectedEggIndex === index) ? null : index;
                    renderEggInventory();
                };
                inventory.appendChild(eggDiv);
            });
        }
        
        function placeEggInIncubator(incubatorIndex) {
            if (gameState.selectedEggIndex === null) {
                showMessage("Select an egg from your inventory first.");
                return;
            }
            if (gameState.incubators[incubatorIndex] !== null) {
                showMessage("This incubator is already in use.");
                return;
            }
            
            const eggToPlace = gameState.eggs.splice(gameState.selectedEggIndex, 1)[0];
            eggToPlace.hatchTime = Date.now() + EGG_DATA[eggToPlace.type].time;
            gameState.incubators[incubatorIndex] = eggToPlace;
            gameState.selectedEggIndex = null;
            updateUI();
        }

        function selectRarity(chances) {
            let rand = Math.random();
            let cumulative = 0;
            for (const rarity in chances) {
                cumulative += chances[rarity];
                if (rand < cumulative) {
                    return rarity;
                }
            }
            return Object.keys(chances)[0]; // Fallback
        }

        function hatchEgg(incubatorIndex) {
            const egg = gameState.incubators[incubatorIndex];
            if (!egg) return;

            const eggData = EGG_DATA[egg.type];
            const chosenRarity = selectRarity(eggData.chances);
            const petPool = Object.keys(PETS).filter(p => PETS[p].rarity === chosenRarity);
            
            if (petPool.length === 0) {
                 console.error(`No pets found for rarity: ${chosenRarity}`);
                 gameState.incubators[incubatorIndex] = null; // Clear incubator
                 showMessage("Hatching failed: no pets of the rolled rarity exist.");
                 updateUI();
                 return;
            }

            const chosenPet = petPool[Math.floor(Math.random() * petPool.length)];
            gameState.incubators[incubatorIndex] = null;
            showHatchAnimation(Object.keys(PETS), chosenPet);
        }
        
        function showHatchAnimation(petPool, chosenPet) {
            modal.classList.remove('hidden');
            modal.classList.add('flex');
            modalContent.innerHTML = `<h2 class="text-3xl mb-4">Hatching...</h2><div id="hatch-spinner" class="text-4xl h-20"></div><div id="hatch-result"></div>`;
            
            const spinner = document.getElementById('hatch-spinner');
            let spinCount = 0;
            const interval = setInterval(() => {
                spinner.textContent = petPool[Math.floor(Math.random() * petPool.length)];
                spinCount++;
                if (spinCount > 30) {
                    clearInterval(interval);
                    spinner.textContent = chosenPet;
                    spinner.classList.add('text-black');
                    document.getElementById('hatch-result').innerHTML = `<p class="text-xl mt-2">Rarity: ${PETS[chosenPet].rarity}</p><button id="confirm-hatch-btn" class="action-button p-2 mt-4">Confirm</button>`;
                    document.getElementById('confirm-hatch-btn').onclick = () => finishHatching(chosenPet);
                }
            }, 100);
        }

        function finishHatching(petName) {
            modal.classList.add('hidden');
            modal.classList.remove('flex');

            if (gameState.discoveredPets[petName]) {
                gameState.pearls += 10;
                showMessage(`You already have a ${petName}. You received 10 pearls.`);
            } else {
                gameState.discoveredPets[petName] = true;
                showMessage(`Congratulations! You hatched a new pet: ${petName}!`);
            }
            updateUI();
        }

        function renderPetCatalog() {
            const catalog = document.getElementById('pet-catalog');
            catalog.innerHTML = '';
            RARITIES.forEach(rarity => {
                const rarityPets = Object.entries(PETS).filter(([_, data]) => data.rarity === rarity);
                if (rarityPets.length === 0) return;

                const raritySection = document.createElement('div');
                raritySection.innerHTML = `<h4 class="text-lg text-gray-600">${rarity} (${rarityPets.length})</h4>`;
                const grid = document.createElement('div');
                grid.className = 'flex flex-wrap gap-2';
                
                rarityPets.forEach(([name, data]) => {
                    const isDiscovered = gameState.discoveredPets[name];
                    const isEquipped = gameState.equippedPets.includes(name);
                    const petSquare = document.createElement('div');
                    petSquare.className = 'pet-square relative flex items-center justify-center';
                    if (!isDiscovered) petSquare.classList.add('blurred');
                    
                    petSquare.innerHTML = `
                        <span class="text-2xl">${getPetIcon(name)}</span>
                        ${isDiscovered ? `
                        <div class="pet-tooltip">
                            <p class="font-bold text-lg">${name}</p>
                            <p class="text-sm text-gray-700 mb-2">${data.perk}</p>
                            <button class="action-button p-1 w-full text-sm" onclick="event.stopPropagation(); toggleEquipPet('${name}')">${isEquipped ? 'Unequip' : 'Equip'}</button>
                        </div>` : ''}
                        ${isEquipped ? '<div class="absolute top-0 right-0 w-3 h-3 bg-black rounded-full"></div>' : ''}
                    `;
                    grid.appendChild(petSquare);
                });
                raritySection.appendChild(grid);
                catalog.appendChild(raritySection);
            });
        }
        
        function toggleEquipPet(petName) {
            const isEquipped = gameState.equippedPets.includes(petName);
            if (isEquipped) {
                gameState.equippedPets = gameState.equippedPets.filter(p => p !== petName);
            } else {
                if (gameState.equippedPets.length >= 2) {
                    showMessage("You can only equip 2 pets at a time.");
                    return;
                }
                gameState.equippedPets.push(petName);
            }
            renderPetCatalog();
        }

        // --- UTILITY FUNCTIONS ---
        function showMessage(message, duration = 3000) {
            modal.classList.remove('hidden');
            modal.classList.add('flex');
            modalContent.innerHTML = `<p class="text-2xl">${message}</p>`;
            setTimeout(() => {
                if (modalContent.innerHTML.includes(message)) {
                    modal.classList.add('hidden');
                    modal.classList.remove('flex');
                }
            }, duration);
        }

        // --- EVENT LISTENERS & INITIALIZATION ---
        function init() {
            loadGame();
            nurseryTabBtn.onclick = () => switchTab('nursery');
            merchantTabBtn.onclick = () => switchTab('merchant');
            
            setInterval(() => {
                if (gameState.currentScreen === 'nursery') {
                    renderIncubators();
                }
            }, 1000);
            
            switchTab('nursery'); // Start on the nursery tab
        }

        // Start the game
        init();
    </script>
</body>
</html>

    
    <!-- New modal for the swim game results -->
    <div id="swim-result-modal" class="modal">
        <div id="swim-result-modal-content" class="modal-content">
            <!-- Content will be dynamically populated -->
        </div>
    </div>
    
    <!-- New modal for solo events -->
    <div id="solo-event-modal" class="modal">
        <div id="solo-event-modal-content" class="modal-content">
            <!-- Content will be dynamically populated -->
            <button id="solo-event-continue-button" class="modal-button">Continue</button>
        </div>
    </div>

    <!-- New modal container for landmark and dive inventory -->
    <div id="landmark-modal-container" class="modal">
        <div id="landmark-modal-content" class="modal-content">
            <!-- Dynamic landmark content -->
        </div>
        <div id="dive-inventory-popup">
            <h3>Dive Inventory</h3>
            <div id="dive-inventory-popup-content"></div>
        </div>
    </div>
    
    <!-- New modal for Sunken Battleship choice -->
    <div id="battleship-choice-modal" class="modal">
        <div class="modal-content">
            <h3>Sunken Battleship</h3>
            <p>You've found a cache of rare materials and schematics for a powerful weapon. You can only take one thing.</p>
            <div id="battleship-choices" style="display: flex; flex-direction: column; gap: 0.5rem; margin: 1rem 0;">
                <button class="modal-button battleship-choice-button" data-item="Tungsten">Take 1 Tungsten</button>
                <button class="modal-button battleship-choice-button" data-item="Kevlar">Take 1 Kevlar</button>
                <button class="modal-button battleship-choice-button" data-item="Reinforced Wood">Take 1 Reinforced Wood</button>
                <button class="modal-button battleship-choice-button" data-item="Reinforced Stone">Take 1 Reinforced Stone</button>
            </div>
        </div>
    </div>

    
    <!-- New modal for combat loot -->
    <div id="loot-modal" class="modal">
        <div id="loot-modal-content" class="modal-content">
            <!-- Dynamic content -->
        </div>
    </div>

    <!-- New modal for AFK timer -->
    <div id="afk-modal" class="modal">
        <div class="modal-content">
            <p>Waiting for player to resume game...</p>
            <button id="resume-game-button" class="modal-button">Resume</button>
        </div>
    </div>
    
    <!-- New modal for combat -->
    <div id="combat-modal" class="modal">
        <div class="modal-content">
            <h2>Enemy Encounter!</h2>
            <div class="combat-arena">
                <div class="combatant" id="combat-player">
                    <div class="combatant-char">@</div>
                    <div>Player</div>
                    <div class="health-bar-container">
                        <div id="player-health-bar" class="health-bar"></div>
                    </div>
                    <div id="player-health-text">15 / 15</div>
                </div>
                <div class="combatant" id="combat-enemy">
                    <div class="combatant-char">?</div>
                    <div id="enemy-name">Enemy</div>
                    <div class="health-bar-container">
                        <div id="enemy-health-bar" class="health-bar"></div>
                    </div>
                    <div id="enemy-health-text">X / X</div>
                </div>
            </div>
            <div id="combat-actions" class="combat-actions">
                <button id="fist-attack-button" class="game-button">Fists (1-2) [1]</button>
                <button id="spear-attack-button" class="game-button" style="display: none;">Shank (2-4) [2]</button>
                <button id="tangle-button" class="game-button" style="display: none;">Tangle [3]<div id="cooldown-bar-tangle" class="cooldown-bar"></div></button>
                <button id="harpoon-attack-button" class="game-button" style="display: none;">Harpoon (5) [4]<div id="cooldown-bar-harpoon" class="cooldown-bar"></div></button>
                <button id="heal-button" class="game-button">Heal (+3) [0]
                    <div id="cooldown-bar-heal" class="cooldown-bar"></div>
                </button>
                <button id="flee-button" class="game-button">Flee (30%)</button>
            </div>
            <div id="combat-log"></div>
        </div>
    </div>
    
    <!-- Wave Animation Container -->
    <div class="ocean">
        <div class="wave"></div>
        <div class="wave"></div>
        <div class="wave"></div>
    </div>


    <script>
        // Get references to the HTML elements
        const gameContainerWrapper = document.querySelector('.game-container-wrapper');
        const mainButton = document.getElementById('main-button');
        const buttonText = document.getElementById('button-text');
        const cooldownBar = document.getElementById('cooldown-bar');
        const narrativePanel = document.getElementById('narrative-lines-container');
        const inventoryPanel = document.getElementById('inventory-panel');
        const knifeItem = document.getElementById('knife-item');
        const axeItem = document.getElementById('axe-item');
        const woodItem = document.getElementById('wood-item');
        const woodCountSpan = document.getElementById('wood-count');
        const gameScreen = document.getElementById('game-screen');
        const resourceGatheringScreen = document.getElementById('resource-gathering-screen');
        const craftingScreen = document.getElementById('crafting-screen');
        const workersScreen = document.getElementById('workers-screen');
        const gameTab = document.getElementById('game-tab');
        const resourceTab = document.getElementById('resource-tab');
        const craftingTab = document.getElementById('crafting-tab');
        const craftingIndicator = document.getElementById('crafting-indicator');
        const achievementsTab = document.getElementById('achievements-tab');
        const workersTab = document.getElementById('workers-tab');
        const workersIndicator = document.getElementById('workers-indicator');
        const navigationTabs = document.querySelector('.navigation-tabs');
        const craftWoodpileButton = document.getElementById('craft-woodpile-button');
        const woodpileRecipe = document.getElementById('woodpile-recipe');
        const cutRopeModal = document.getElementById('cut-rope-modal');
        const cutRopeButton = document.getElementById('cut-rope-button');
        const lightFireModal = document.getElementById('light-fire-modal');
        const lightFireButton = document.getElementById('light-fire-button');
        const unlitWoodpileItem = document.getElementById('unlit-woodpile-item');
        const flammableLiquidItem = document.getElementById('flammable-liquid-item');
        const matchItem = document.getElementById('match-item');
        const achievementsScreen = document.getElementById('achievements-screen');
        const achievementContainer = document.getElementById('achievement-container');
        const achievementListContainer = document.getElementById('achievement-list-container');
        const backButton = document.getElementById('back-button');
        const gatherWoodButton = document.getElementById('gather-wood-button');
        const gatherCooldownBar = document.getElementById('cooldown-bar-gather');
        const swimButton = document.getElementById('swim-button');
        const swimCooldownBar = document.getElementById('cooldown-bar-swim');
        const swimResultModal = document.getElementById('swim-result-modal');
        const swimResultModalContent = document.getElementById('swim-result-modal-content');
        const stoneItem = document.getElementById('stone-item');
        const stoneCountSpan = document.getElementById('stone-count');
        const fabricItem = document.getElementById('fabric-item');
        const fabricCountSpan = document.getElementById('fabric-count');
        const ropeItem = document.getElementById('rope-item');
        const ropeCountSpan = document.getElementById('rope-count');
        const metallicScrapItem = document.getElementById('metallic-scrap-item');
        const metallicScrapCountSpan = document.getElementById('metallic-scrap-count');
        const bandageItem = document.getElementById('bandage-item');
        const bandageCountSpan = document.getElementById('bandage-count');
        const newPersonModal = document.getElementById('new-person-modal');
        const takeInPersonButton = document.getElementById('take-in-person-button');
        const newPeopleModal = document.getElementById('new-people-modal');
        const takeInPeopleButton = document.getElementById('take-in-people-button');
        const totalWorkersCountSpan = document.getElementById('total-workers-count');
        const woodGatherersCountSpan = document.getElementById('wood-gatherers-count');
        const swimmersCountSpan = document.getElementById('swimmers-count');
        const idleWorkersCountSpan = document.getElementById('idle-workers-count');
        const assignWoodGathererButton = document.getElementById('assign-wood-gatherer');
        const assignSwimmerButton = document.getElementById('assign-swimmer');
        const recallWoodGathererButton = document.getElementById('recall-wood-gatherer');
        const recallSwimmerButton = document.getElementById('recall-swimmer');
        
        // New elements for progression screen
        const progressionButton = document.getElementById('progression-button');
        const progressionScreen = document.getElementById('progression-screen');
        const progressionBackButton = document.getElementById('progression-back-button');
        const section2 = document.getElementById('section-2');
        const section3 = document.getElementById('section-3');
        
        // Music control elements
        const musicControlBtn = document.getElementById('music-control');
        const backgroundMusic = document.getElementById('background-music');
        const clickSound = document.getElementById('click-sound');
        
        // New elements for the loading screen
        const swimLoadingScreen = document.getElementById('swim-loading-screen');
        const loadingBar = document.getElementById('loading-bar');
        const bouncingText = document.querySelector('.bouncing-text');
        
        // New elements for the solo event modal
        const soloEventModal = document.getElementById('solo-event-modal');
        const soloEventModalContent = document.getElementById('solo-event-modal-content');
        
        // New elements for diving
        const diveTab = document.getElementById('dive-tab');
        const diveIndicator = document.getElementById('dive-indicator');
        const diveScreen = document.getElementById('dive-screen');
        const divePrepArea = document.getElementById('dive-prep-area');
        const toolSelectionContainer = document.getElementById('tool-selection-container');
        const beginDiveButton = document.getElementById('begin-dive-button');
        const diveCooldownBar = document.getElementById('cooldown-bar-dive');
        const diveMinigameArea = document.getElementById('dive-minigame-area');
        const minigameGrid = document.getElementById('minigame-grid');
        const movesLeftSpan = document.getElementById('moves-left');
        const diveHealthText = document.getElementById('dive-health-text');
        const divePlayerHealthBar = document.getElementById('dive-player-health-bar');
        const useBandageButton = document.getElementById('use-bandage-button');
        const buildHabitatButton = document.getElementById('build-habitat-button');
        const diveBandageCountSpan = document.getElementById('dive-bandage-count');
        const divingGearItem = document.getElementById('diving-gear-item');
        const spearItem = document.getElementById('spear-item');
        const airTankItem = document.getElementById('air-tank-item');
        const pearlItem = document.getElementById('pearl-item');
        const pearlCountSpan = document.getElementById('pearl-count');
        const harpoonGunItem = document.getElementById('harpoon-gun-item');
        const harpoonAmmoItem = document.getElementById('harpoon-ammo-item');
        const harpoonAmmoCountSpan = document.getElementById('harpoon-ammo-count');
        const fishingNetItem = document.getElementById('fishing-net-item');
        const diveInventoryDisplay = document.getElementById('dive-inventory-display');
        const diveBagItem = document.getElementById('dive-bag-item');
        const furnaceItem = document.getElementById('furnace-item');
        const screwsItem = document.getElementById('screws-item');
        const screwsCountSpan = document.getElementById('screws-count');
        
        // New elements for deep diving
        const deepDiveTab = document.getElementById('deep-dive-tab');
        const deepDiveScreen = document.getElementById('deep-dive-screen');
        const deepDivePrepArea = document.getElementById('deep-dive-prep-area');
        const deepToolSelectionContainer = document.getElementById('deep-tool-selection-container');
        const beginDeepDiveButton = document.getElementById('begin-deep-dive-button');
        const deepDiveCooldownBar = document.getElementById('cooldown-bar-deep-dive');
        const deepDiveMinigameArea = document.getElementById('deep-dive-minigame-area');
        const deepMinigameGrid = document.getElementById('deep-minigame-grid');
        const deepMovesLeftSpan = document.getElementById('deep-moves-left');
        const deepDiveHealthText = document.getElementById('deep-dive-health-text');
        const deepDivePlayerHealthBar = document.getElementById('deep-dive-player-health-bar');
        const deepUseBandageButton = document.getElementById('deep-use-bandage-button');
        const deepDiveBandageCountSpan = document.getElementById('deep-dive-bandage-count');
        const deepDiveInventoryDisplay = document.getElementById('deep-dive-inventory-display');
        const complexDivingGearItem = document.getElementById('complex-diving-gear-item');

        
        // New elements for commands
        const commandsButton = document.getElementById('commands-button');
        const commandsScreen = document.getElementById('commands-screen');
        const adminCodeInput = document.getElementById('admin-code-input');
        const adminSubmitButton = document.getElementById('admin-submit-button');
        const adminStatus = document.getElementById('admin-status');
        const adminControlsArea = document.getElementById('admin-controls-area');
        const freeCraftingToggle = document.getElementById('free-crafting-toggle');
        const fastCooldownsToggle = document.getElementById('fast-cooldowns-toggle');
        const infiniteMovesToggle = document.getElementById('infinite-moves-toggle');
        const godHealthToggle = document.getElementById('god-health-toggle');
        const noEnemiesToggle = document.getElementById('no-enemies-toggle');
        const commandsBackButton = document.getElementById('commands-back-button');
        
        // New landmark and combat elements
        const landmarkModalContainer = document.getElementById('landmark-modal-container');
        const landmarkModalContent = document.getElementById('landmark-modal-content');
        const diveInventoryPopupContent = document.getElementById('dive-inventory-popup-content');
        const lootModal = document.getElementById('loot-modal');
        const lootModalContent = document.getElementById('loot-modal-content');
        const combatModal = document.getElementById('combat-modal');
        const playerHealthBar = document.getElementById('player-health-bar');
        const playerHealthText = document.getElementById('player-health-text');
        const enemyName = document.getElementById('enemy-name');
        const enemyHealthBar = document.getElementById('enemy-health-bar');
        const enemyHealthText = document.getElementById('enemy-health-text');
        const fistAttackButton = document.getElementById('fist-attack-button');
        const spearAttackButton = document.getElementById('spear-attack-button');
        const harpoonAttackButton = document.getElementById('harpoon-attack-button');
        const healButton = document.getElementById('heal-button');
        const healCooldownBar = document.getElementById('cooldown-bar-heal');
        const combatLog = document.getElementById('combat-log');
        const fleeButton = document.getElementById('flee-button');
        const battleshipChoiceModal = document.getElementById('battleship-choice-modal');

        
        // Purity Crafting elements
        const purityCraftingContainer = document.getElementById('purity-crafting-container');
        const purityRecipesContainer = document.getElementById('purity-recipes-container');
        const tungstenItem = document.getElementById('tungsten-item');
        const tungstenCountSpan = document.getElementById('tungsten-count');
        const kevlarItem = document.getElementById('kevlar-item');
        const kevlarCountSpan = document.getElementById('kevlar-count');
        const reinforcedStoneItem = document.getElementById('reinforced-stone-item');
        const reinforcedStoneCountSpan = document.getElementById('reinforced-stone-count');
        const reinforcedWoodItem = document.getElementById('reinforced-wood-item');
        const reinforcedWoodCountSpan = document.getElementById('reinforced-wood-count');
        const kevlarArmourItem = document.getElementById('kevlar-armour-item');


        // Save/Load elements
        const saveButton = document.getElementById('save-button');
        const saveLoadScreen = document.getElementById('save-load-screen');
        const exportTextarea = document.getElementById('export-textarea');
        const importTextarea = document.getElementById('import-textarea');
        const importButton = document.getElementById('import-button');
        const saveLoadBackButton = document.getElementById('save-load-back-button');
        const importConfirmModal = document.getElementById('import-confirm-modal');
        const confirmImportButton = document.getElementById('confirm-import-button');
        const cancelImportButton = document.getElementById('cancel-import-button');
        
        // Tangle button
        const tangleButton = document.getElementById('tangle-button');
        const tangleCooldownBar = document.getElementById('cooldown-bar-tangle');
        
        // AFK elements
        const afkModal = document.getElementById('afk-modal');
        const resumeGameButton = document.getElementById('resume-game-button');

        // Statistics elements
        const statisticsButton = document.getElementById('statistics-button');
        const statisticsScreen = document.getElementById('statistics-screen');
        const statsContainer = document.getElementById('stats-container');
        const statisticsBackButton = document.getElementById('statistics-back-button');
        const bestiaryButton = document.getElementById('bestiary-button');
        const bestiaryScreen = document.getElementById('bestiary-screen');
        const bestiaryContainer = document.getElementById('bestiary-container');
        const bestiaryBackButton = document.getElementById('bestiary-back-button');
        
        // Timer elements
        const speedrunTimer = document.getElementById('speedrun-timer');
        
        // Nursery and Merchant elements
        const nurseryTab = document.getElementById('nursery-tab');
        const merchantTab = document.getElementById('merchant-tab');
        const nurseryScreen = document.getElementById('nursery-screen');
        const merchantScreen = document.getElementById('merchant-screen');
        const incubatorSlotsContainer = document.getElementById('incubator-slots');
        const eggInventoryDisplay = document.getElementById('egg-inventory-display');
        const petCatalogContainer = document.getElementById('pet-catalog-container');
        const pearlDisplay = document.getElementById('pearl-display');
        const merchantEggsContainer = document.getElementById('merchant-eggs');
        const hatchModal = document.getElementById('hatch-modal');
        const wheelContainer = document.getElementById('wheel-container');
        const hatchResultButton = document.getElementById('hatch-result-button');



        
        // State variables to track game progress
        let searchCount = 0;
        let isFree = false;
        let woodCount = 0;
        let stoneCount = 0;
        let fabricCount = 0;
        let ropeCount = 0;
        let metallicScrapCount = 0;
        let bandageCount = 0;
        let screwsCount = 0;
        let axeheadFound = false;
        let hasUnlitWoodpile = false;
        let hasFlammableLiquid = false;
        let hasMatch = false;
        let isMusicPlaying = false;
        let selectedItems = new Set();
        let unlockedAchievements = new Set();
        let encounteredEnemies = new Set();
        
        // New state for story progression and events
        let isSection2 = false;
        let isSection3 = false;
        let isPlayerSwimming = false;
        let soloEventTimer = null;
        let narrativeTimer = null;
        let eventQueue = [];
        let popupQueue = [];
        let hasShownFireHint = false;
        let hasFoundRepairShip = false;
        let hasFoundBattleship = false;
        
        // New state for workers and shelters
        let totalWorkers = 0;
        let idleWorkers = 0;
        let woodGatherers = 0;
        let swimmers = 0;
        const shelterCosts = [10, 20, 30, 40, 50];
        let sheltersBuilt = 0;
        let reinforcedSheltersBuilt = 0;
        const MAX_REINFORCED_SHELTERS = 5;
        
        // New state for diving
        let isPlayerDiving = false;
        let hasDivingGear = false;
        let hasSpear = false;
        let hasBetterAirTank = false;
        let hasDiveBag = false;
        let hasFurnace = false;
        let habitatsBuilt = 0;
        const MAX_HABITATS = 5;
        let placedHabitats = []; // [{x, y, usedThisDive}]
        let pearlCount = 0;
        let hasHarpoonGun = false;
        let harpoonAmmoCount = 0;
        let hasFishingNet = false;
        let diveMoves = 20;
        let diveInventory = []; // Array of up to 4 objects: { item, quantity }
        let DIVE_INV_SLOTS = 4;
        const DIVE_STACK_LIMIT = 5;
        let diveTools = [];
        let diveBandages = 0;
        let minigameMap = [];
        let playerPosition = { x: 0, y: 0 };
        let playerPath = []; // To track path for safe routes
        let safePaths = []; // To store successful paths
        const MAP_SIZE = 51; // Increased map size
        let rareEnemyPathwayChance = 0.05; // Base 5% chance, increases
        
        // New state for deep diving
        let isPlayerDeepDiving = false;
        let hasComplexDivingGear = false;
        let deepDiveMoves = 5;
        let deepDiveInventory = [];
        let deepDiveTools = [];
        let deepDiveBandages = 0;
        let deepMinigameMap = [];
        let deepPlayerPosition = { x: 0, y: 0 };
        
        // New state for deep dive resources
        let coralFragmentsCount = 0, hydrothermalSludgeCount = 0, bioluminescenceSacCount = 0, electroluminescentFilamentCount = 0, chitinousPlatingCount = 0, nautilusShellCount = 0, etherealVisceraCount = 0, hydrothermalVentMineralCount = 0, blackIceCoreCount = 0, giantSquidBeakCount = 0, abyssalPearlCount = 0, mythicToothCount = 0, boneFragmentCount = 0;

        
        // New state for combat
        let DEFAULT_MAX_PLAYER_HEALTH = 15;
        let MAX_PLAYER_HEALTH = DEFAULT_MAX_PLAYER_HEALTH;
        let playerHealth = MAX_PLAYER_HEALTH;
        let enemyHealth = 0;
        let maxEnemyHealth = 0;
        let currentEnemy = null;
        let enemyAttackInterval = null;
        let isPlayerInCombat = false;
        let isHealOnCooldown = false;
        let isPlayerStunned = false;
        let isFistDisabled = false;
        let isEnemyStunned = false;
        let enemyStunTimeout = null;
        let combatTurnCounter = 0;
        let playerDoT = { damage: 0, duration: 0 }; // For Giant Phantom Jelly

        // New state for admin commands
        let isAdminMode = false;
        let isFreeCrafting = false;
        let isCooldownHacked = false;
        let hasInfiniteMoves = false;
        let hasGodHealth = false;
        let noEnemies = false;
        let unlockedAdminCommands = new Set();
        
        // New state for purity materials
        let tungstenCount = 0;
        let kevlarCount = 0;
        let reinforcedStoneCount = 0;
        let reinforcedWoodCount = 0;
        let hasKevlarArmour = false;
        
        // New state for Nursery/Pets
        let hasUnlockedNursery = false;
        let eggInventory = []; // { type, id }
        const MAX_INCUBATORS = 2;
        let incubators = Array(MAX_INCUBATORS).fill(null); // { egg, hatchTime }
        let ownedPets = new Set();
        let equippedPets = [];
        const MAX_EQUIPPED_PETS = 2;


        
        // Cooldown durations & Timers
        const searchCooldownDuration = 700;
        let swimCooldownDuration = 90000;
        let gatherCooldownDuration = 4000;
        const diveCooldownDuration = 150000;
        const deepDiveCooldownDuration = 120000; // 2 minutes
        const healCooldownDuration = 5000;
        const tangleCooldownDuration = 8000;
        const harpoonCooldownDuration = 3000;
        const swimLoadingDuration = 10000;
        const cooldownReductionDuration = 120000;
        let isCooldownReduced = false;
        let initialSwimCooldown = 90000;
        let initialGatherCooldown = 4000;
        
        // AFK Timer
        let afkTimer = null;
        const AFK_TIMEOUT = 300000; // 5 minutes
        let isGamePaused = false;
        
        // Worker intervals
        const workerWoodInterval = 20000;
        const workerSwimInterval = 20000;
        let woodWorkerIntervals = [];
        let swimWorkerIntervals = [];
        
        // Statistics tracking
        let gameStats = {
            totalWoodCollected: 0,
            totalStoneCollected: 0,
            totalFabricCollected: 0,
            totalRopeCollected: 0,
            totalMetallicScrapCollected: 0,
            totalDives: 0,
            totalDeepDives: 0,
            enemiesSlain: {}
        };
        
        // Timer state
        let gameTime = 0;
        let timerInterval = null;


        // Narrative text logs
        const narrativeLogTied = [
            "Your hands are tied to a rope. The rope feels coarse and rough.",
            "You try to move your hands, but the rope is too tight. You feel a small, hard object on the deck.",
            "You try to reach for the object with your foot. You can just barely touch it.",
            "The object is a small, rusted knife. The blade is dull, but maybe it can be used to cut the rope.",
            "You try to grab the knife with your toes, but you can't.",
            "You manage to wiggle the knife closer with your feet.",
            "Success! You have the knife. Now to get free.",
            "You have found a rusted knife. Use it to cut the ropes?",
        ];
        
        const narrativeLogFree = [
            "You are free now. You look around, and all you see is an endless, black ocean.",
            "You notice some wooden crates floating in the water. Maybe you can grab some wood to build a fire.",
            "You see something glinting on the deck. It is a rusted axehead, without a handle. It's too heavy to wield.",
            "You continue searching the boat, but find nothing else of interest."
        ];
        
        const narrativeLogAfterWoodpile = [
            "You continue to search the boat. It feels like you have searched every inch.",
            "You hear a sloshing sound, and you see a small puddle of thick, black oil on the deck. It seems to have leaked from a barrel.",
            "The barrel is empty now. A cup of flammable liquid is all that remains."
        ];

        const narrativeLogAfterLiquid = [
            "You search the boat again, but find nothing new of interest.",
            "Your fingers brush against a small, damp cardboard box. Inside, there is one single match, thankfully dry."
        ];
        
        // Narrative text for solo events
        const section2NarrativeLines = [
            "A chilling gust of wind blows past you, making the boat creak ominously. You hug yourself for warmth.",
            "You spot something large moving in the water, but it disappears into the murky depths before you can get a good look.",
            "The waves are getting choppier. The boat sways violently, and you have to grab onto the mast to steady yourself.",
            "You hear a faint, distant melody carried on the wind. It sounds like a song of loss, and it sends a shiver down your spine."
        ];

        // Achievement data
        const achievements = {
            getting_free: { title: "Getting Free", description: "You cut the ropes that bound you.", rarity: "Common" },
            axe_without_handle: { title: "Axe without the Handle", description: "You found a heavy, rusted axehead.", rarity: "Common" },
            chop_chop_chop: { title: "Chop, Chop, Chop", description: "You gathered 5 pieces of wood.", rarity: "Common" },
            woodpile_obtained: { title: "Woodpile Obtained", description: "You crafted a bundle of wood for a fire.", rarity: "Common" },
            reeking_of_petroleum: { title: "Reeking of Petroleum", description: "You found a tiny cup of thick black oil.", rarity: "Common" },
            one_and_only: { title: "One and Only", description: "You found a single match with one in the matchbox.", rarity: "Common" },
            fire_lit: { title: "A Warm Glow", description: "You lit the fire and survived the cold.", rarity: "Rare" },
            first_swim: { title: "First Swim", description: "You bravely swam into the dark ocean.", rarity: "Common" },
            section_2_beginning: { title: "Section 2: Beginning to Begin", description: "You've successfully returned from your first swim.", rarity: "Rare" },
            first_member: { title: "A Helping Hand", description: "You welcomed another person to your boat.", rarity: "Common" },
            helpful_hands: { title: "Helpful Hands", description: "You received materials from one of your workers.", rarity: "Common" },
            deep_diver: { title: "Deep Diver", description: "You crafted diving gear to explore the depths.", rarity: "Rare" },
            deep_sea_dweller: { title: "Deep-Sea Dweller", description: "You placed your first underwater habitat.", rarity: "Rare" },
            abyss_gazer: { title: "Abyss Gazer", description: "You unlocked the Deep Dive tab.", rarity: "Legendary" },
            first_kill_deep: { title: "First Blood of the Abyss", description: "You killed your first deep-sea creature.", rarity: "Legendary" },
            god_mode: { title: "You Little Sh*t", description: "You have enabled free crafting.", rarity: "Secret" },
            afk_lol: { title: "You didn’t think that actually worked…", description: "You triggered the AFK timer.", rarity: "Secret" },
            welcome_back: { title: "Welcome Back", description: "You loaded a previous save file.", rarity: "Legendary" }
        };

        // Function to unlock an achievement and display the pop-up
        function unlockAchievement(achievementId) {
            if (unlockedAchievements.has(achievementId)) {
                return; // Already unlocked, do nothing
            }
            unlockedAchievements.add(achievementId);
            
            const achievement = achievements[achievementId];
            const popup = document.createElement('div');
            popup.classList.add('achievement-popup');
            popup.innerHTML = `
                <div class="achievement-title">${achievement.title} Unlocked!</div>
                <div class="achievement-description">${achievement.description}</div>
                <div class="achievement-rarity">${achievement.rarity}</div>
            `;
            achievementContainer.appendChild(popup);
            
            // Remove the popup after the animation finishes
            setTimeout(() => {
                popup.remove();
            }, 4000); // Wait for the 4s animation to complete
        }

        // Function to update the achievements screen with rarity sections
        function updateAchievementScreen() {
            // Clear all sections first
            document.querySelectorAll('.achievement-rarity-section').forEach(section => {
                const title = section.querySelector('h2').outerHTML;
                section.innerHTML = title;
            });

            let hasAnyAchievements = false;
            for (const id in achievements) {
                if (unlockedAchievements.has(id)) {
                    hasAnyAchievements = true;
                    const achievement = achievements[id];
                    const section = Array.from(document.querySelectorAll('.achievement-rarity-section h2')).find(h2 => h2.textContent.toLowerCase() === achievement.rarity.toLowerCase()).parentElement;
                    
                    if (section) {
                        const item = document.createElement('div');
                        item.classList.add('achievement-list-item', 'unlocked');
                        item.innerHTML = `
                            <div class="achievement-title">${achievement.title}</div>
                            <div class="achievement-description">${achievement.description}</div>
                        `;
                        section.appendChild(item);
                    }
                }
            }

            if (!hasAnyAchievements) {
                const message = document.createElement('p');
                message.textContent = "You haven't unlocked any achievements yet. Keep playing!";
                document.querySelector('.achievement-rarity-section').appendChild(message);
            }
        }
        
        // Function to add a narrative line to the panel with a fade-in effect
        function addNarrativeLine(text) {
            const p = document.createElement('p');
            p.classList.add('narrative-line');
            p.textContent = text;
            narrativePanel.appendChild(p);

            // Keep only the last 6 lines to prevent the panel from getting too long
            const lines = narrativePanel.querySelectorAll('.narrative-line');
            if (lines.length > 6) {
                // Fade out the oldest line
                lines[0].classList.add('fade-out');
                // Remove the line after the transition
                setTimeout(() => {
                    lines[0].remove();
                }, 1000);
            }

            // Force a reflow to make sure the transition starts
            void p.offsetWidth;
            p.style.opacity = '1';
            p.style.transform = 'translateY(0)';
            
            // Scroll to the bottom of the narrative panel to show new text
            narrativePanel.scrollTop = narrativePanel.scrollHeight;
        }
        
        // --- Cooldown and Timer Management ---
        let activeCooldowns = [];

        function getCooldownDuration(baseDuration) {
            return isCooldownHacked ? baseDuration * 0.15 : baseDuration;
        }

        function startCooldown(cooldownButton, cooldownBar, duration) {
            const finalDuration = getCooldownDuration(duration);
            if (finalDuration <= 0) { // Handle instant cooldown reset
                cooldownButton.classList.remove('disabled');
                if (cooldownBar) cooldownBar.style.width = '0%';
                return;
            }

            cooldownButton.classList.add('disabled');
            if (cooldownBar) cooldownBar.style.width = '100%';
            
            let timeLeft = finalDuration;
            const interval = 100;
            
            const timerData = {
                button: cooldownButton,
                bar: cooldownBar,
                duration: finalDuration,
                timeLeft: timeLeft,
                isPaused: false
            };

            const timerId = setInterval(() => {
                if (timerData.isPaused || isGamePaused) return; // Check global pause as well

                timerData.timeLeft -= interval;
                const percentage = (timerData.timeLeft / timerData.duration) * 100;
                if(timerData.bar) timerData.bar.style.width = percentage + '%';

                if (timerData.timeLeft <= 0) {
                    clearInterval(timerId);
                    timerData.button.classList.remove('disabled');
                    if(timerData.bar) timerData.bar.style.width = '0%';
                    activeCooldowns = activeCooldowns.filter(t => t.id !== timerId);
                }
            }, interval);
            
            timerData.id = timerId;
            activeCooldowns.push(timerData);
        }
        
        // --- Modal and Popup Management ---
        function showModal(modalFunction) {
            if (isGamePaused) {
                popupQueue.push(modalFunction);
            } else {
                modalFunction();
            }
        }

        function processPopupQueue() {
            if (popupQueue.length > 0) {
                const nextPopup = popupQueue.shift();
                nextPopup();
            }
        }

        // Function to handle the new person recruitment
        function recruitNewPerson() {
            newPersonModal.style.display = 'none';
            totalWorkers++;
            idleWorkers++;
            updateWorkerStats();
            
            if (totalWorkers === 1) {
                unlockAchievement('first_member');
                workersTab.style.display = 'block';
                workersIndicator.style.display = 'block';
                addNarrativeLine("You have a new crew member! You can now assign tasks to them in the new 'Workers' tab.");
            }
            
            updateShelterRecipe();
        }
        
        function recruitNewPeople() {
            newPeopleModal.style.display = 'none';
            totalWorkers += 2;
            idleWorkers += 2;
            updateWorkerStats();
            updateReinforcedShelterRecipe();
        }
        
        // Function to handle shelter crafting
        function craftShelter() {
            if (sheltersBuilt >= shelterCosts.length) {
                addNarrativeLine("You have built the maximum number of shelters.");
                return;
            }
            if (sheltersBuilt > totalWorkers) {
                addNarrativeLine("You must wait for a new person to occupy your last shelter before building another.");
                return;
            }

            const cost = shelterCosts[sheltersBuilt];
            if (isFreeCrafting || woodCount >= cost) {
                if (!isFreeCrafting) {
                    woodCount -= cost;
                    woodCountSpan.textContent = woodCount;
                }
                sheltersBuilt++;
                addNarrativeLine(`You've built shelter #${sheltersBuilt}! It cost ${isFreeCrafting ? 0 : cost} wood.`);
                
                setTimeout(() => {
                    showModal(() => newPersonModal.style.display = 'flex');
                }, 10000);
                
                updateShelterRecipe();

            } else {
                addNarrativeLine(`You need ${cost} wood to build the next shelter. You only have ${woodCount}.`);
            }
        }
        
        function craftReinforcedShelter() {
            if (reinforcedSheltersBuilt >= MAX_REINFORCED_SHELTERS) {
                addNarrativeLine("You have built the maximum number of reinforced shelters.");
                return;
            }
            // Check if the last reinforced shelter is occupied
            if (reinforcedSheltersBuilt > 0 && (totalWorkers - 5) / 2 < reinforcedSheltersBuilt) {
                addNarrativeLine("You must wait for new people to occupy your last reinforced shelter before building another.");
                return;
            }

            const woodCost = 25 + (reinforcedSheltersBuilt * 25);
            const reinforcedWoodCost = 2;

            if (isFreeCrafting || (woodCount >= woodCost && reinforcedWoodCount >= reinforcedWoodCost)) {
                if (!isFreeCrafting) {
                    woodCount -= woodCost;
                    reinforcedWoodCount -= reinforcedWoodCost;
                    updateAllResourceCounts();
                }
                reinforcedSheltersBuilt++;
                addNarrativeLine(`You've built reinforced shelter #${reinforcedSheltersBuilt}!`);
                
                const randomDelay = Math.random() * 5000 + 5000; // 5-10 seconds
                setTimeout(() => {
                    showModal(() => newPeopleModal.style.display = 'flex');
                }, randomDelay);
                
                updateReinforcedShelterRecipe();

            } else {
                addNarrativeLine(`You need ${woodCost} Wood and ${reinforcedWoodCost} Reinforced Wood.`);
            }
        }

        
        // Function to dynamically add or update a crafting recipe
        function addCraftingRecipe(name, cost, buttonId, recipeId, clickHandler, containerId = 'crafting-recipes-container', description = null) {
            const container = document.getElementById(containerId);
            let existingRecipe = document.getElementById(recipeId);

            if (!existingRecipe) {
                const newRecipe = document.createElement('div');
                newRecipe.classList.add('crafting-recipe');
                newRecipe.id = recipeId;
                let descriptionHTML = description ? `<p><i>${description}</i></p>` : '';
                newRecipe.innerHTML = `
                    <p>${name}</p>
                    <p>Requires: ${cost}</p>
                    ${descriptionHTML}
                    <button id="${buttonId}" class="game-button">Craft<div class="cooldown-bar"></div></button>
                `;
                container.appendChild(newRecipe);
                document.getElementById(buttonId).addEventListener('click', clickHandler);
            }
        }

        function updateShelterRecipe() {
            const recipeId = 'craft-shelter-recipe';
            let recipe = document.getElementById(recipeId);

            if (sheltersBuilt >= shelterCosts.length) {
                if (recipe) recipe.style.display = 'none';
                updateReinforcedShelterRecipe(); // Show reinforced shelter recipe when max normal shelters are built
                return;
            }
            
            const cost = shelterCosts[sheltersBuilt];
            const name = `Small Shelter #${sheltersBuilt + 1}`;
            const buttonId = `craft-shelter-${sheltersBuilt}`;

            if (!recipe) {
                addCraftingRecipe(name, `${cost} Wood`, buttonId, recipeId, craftShelter);
            } else {
                recipe.style.display = 'block';
                recipe.querySelector('p:first-of-type').textContent = name;
                recipe.querySelector('p:nth-of-type(2)').textContent = `Requires: ${cost} Wood`;
                const oldButton = recipe.querySelector('button');
                oldButton.id = buttonId;
                const newButton = oldButton.cloneNode(true);
                oldButton.parentNode.replaceChild(newButton, oldButton);
                newButton.addEventListener('click', craftShelter);
            }
        }
        
        function updateReinforcedShelterRecipe() {
            const recipeId = 'craft-reinforced-shelter-recipe';
            let recipe = document.getElementById(recipeId);

            if (reinforcedSheltersBuilt >= MAX_REINFORCED_SHELTERS) {
                if (recipe) recipe.style.display = 'none';
                return;
            }
            
            const woodCost = 25 + (reinforcedSheltersBuilt * 25);
            const reinforcedWoodCost = 2;
            const name = `Reinforced Shelter #${reinforcedSheltersBuilt + 1}`;
            const buttonId = `craft-reinforced-shelter-${reinforcedSheltersBuilt}`;

            if (!recipe) {
                addCraftingRecipe(name, `${woodCost} Wood, ${reinforcedWoodCost} Reinforced Wood`, buttonId, recipeId, craftReinforcedShelter);
            } else {
                recipe.style.display = 'block';
                recipe.querySelector('p:first-of-type').textContent = name;
                recipe.querySelector('p:nth-of-type(2)').textContent = `Requires: ${woodCost} Wood, ${reinforcedWoodCost} Reinforced Wood`;
                const oldButton = recipe.querySelector('button');
                oldButton.id = buttonId;
                const newButton = oldButton.cloneNode(true);
                oldButton.parentNode.replaceChild(newButton, oldButton);
                newButton.addEventListener('click', craftReinforcedShelter);
            }
        }


        // The main function to handle game progression
        function progressGame() {
            if (mainButton.classList.contains('disabled')) return;
            startCooldown(mainButton, cooldownBar, searchCooldownDuration);

            if (hasUnlitWoodpile && hasFlammableLiquid && hasMatch && !hasShownFireHint) {
                addNarrativeLine("You have a woodpile, some flammable liquid, and a single match. Try clicking the woodpile and the match in your inventory to light a fire!");
                hasShownFireHint = true;
                return;
            }

            if (!isFree) {
                if (searchCount < narrativeLogTied.length) {
                    addNarrativeLine(narrativeLogTied[searchCount]);
                    if (searchCount === 7) {
                        mainButton.style.display = 'none';
                        inventoryPanel.style.display = 'flex';
                        knifeItem.style.display = 'block';
                    }
                    searchCount++;
                }
            } else if (!axeheadFound) {
                if (searchCount < narrativeLogFree.length) {
                    addNarrativeLine(narrativeLogFree[searchCount]);
                    searchCount++;
                }
                if (searchCount === narrativeLogFree.length) {
                    addNarrativeLine("You have found the axehead. It has no handle. You can use it to break a piece of wood from a crate. The crates are waterlogged, so the wood will not burn on its own.");
                    navigationTabs.style.display = 'flex';
                    inventoryPanel.style.display = 'flex';
                    knifeItem.style.display = 'none';
                    axeItem.style.display = 'block';
                    axeheadFound = true;
                    unlockAchievement('axe_without_handle');
                    mainButton.style.display = 'none';
                    searchCount = 0;
                }
            } else if (!hasUnlitWoodpile) {
                addNarrativeLine("You need to craft a woodpile. Check the 'Resource Gathering' and 'Crafting' tabs.");
            } else if (!hasFlammableLiquid) {
                if (searchCount < narrativeLogAfterWoodpile.length) {
                    addNarrativeLine(narrativeLogAfterWoodpile[searchCount]);
                    if (searchCount === narrativeLogAfterWoodpile.length - 1) {
                        flammableLiquidItem.style.display = 'flex';
                        hasFlammableLiquid = true;
                        unlockAchievement('reeking_of_petroleum');
                        searchCount = 0;
                    }
                    searchCount++;
                } else {
                    addNarrativeLine("You search the boat again, but find nothing new.");
                }
            } else if (!hasMatch) {
                if (searchCount < narrativeLogAfterLiquid.length) {
                    addNarrativeLine(narrativeLogAfterLiquid[searchCount]);
                    if (searchCount === narrativeLogAfterLiquid.length - 1) {
                        matchItem.style.display = 'flex';
                        hasMatch = true;
                        unlockAchievement('one_and_only');
                    }
                    searchCount++;
                } else {
                    addNarrativeLine("You search the boat again, but find nothing new.");
                }
            } else {
                addNarrativeLine("You search the boat again, but find nothing new.");
            }
        }
        
        // Function to handle gathering wood
        function gatherWood() {
            if (gatherWoodButton.classList.contains('disabled')) return;
            addNarrativeLine("You strike the axehead against a crate, splintering a small piece.");
            woodCount++;
            gameStats.totalWoodCollected++;
            woodItem.style.display = 'flex';
            woodCountSpan.textContent = woodCount;
            if (woodCount >= 5) unlockAchievement('chop_chop_chop');
            startCooldown(gatherWoodButton, gatherCooldownBar, gatherCooldownDuration);
        }

        // Function to handle making a fire
        function craftWoodpile() {
            if (hasUnlitWoodpile) {
                addNarrativeLine("You already have an Unlit Woodpile.");
            } else if (isFreeCrafting || woodCount >= 5) {
                addNarrativeLine("You carefully arrange the wood into a bundle for a fire.");
                if (!isFreeCrafting) {
                    woodCount -= 5;
                    woodCountSpan.textContent = woodCount;
                }
                unlitWoodpileItem.style.display = 'flex';
                hasUnlitWoodpile = true;
                unlockAchievement('woodpile_obtained');
                woodpileRecipe.style.display = 'none';
                searchCount = 0;
                switchScreen('game');
                addNarrativeLine("You've crafted an Unlit Woodpile! Now find something to light it. The 'Search' button is available again.");
                mainButton.style.display = 'block';
                buttonText.textContent = 'Search';
            } else {
                addNarrativeLine(`You need 5 wood. You only have ${woodCount}.`);
            }
        }
        
        // Function to handle clicking inventory items
        function handleInventoryClick(item) {
            if (selectedItems.has(item)) {
                selectedItems.delete(item);
                item.classList.remove('selected');
            } else {
                selectedItems.add(item);
                item.classList.add('selected');
            }
            if (selectedItems.has(unlitWoodpileItem) && selectedItems.has(matchItem)) {
                showModal(() => lightFireModal.style.display = 'flex');
            }
        }

        // Function to use the unlit woodpile
        function lightThePile() {
            lightFireModal.style.display = 'none';
            gameContainerWrapper.classList.add('camera-shake');
            setTimeout(() => gameContainerWrapper.classList.remove('camera-shake'), 500);
            addNarrativeLine("You strike the match. The tiny flame sputters to life and catches the oil-soaked wood. The fire dances, a beacon of warmth. You can now swim for materials.");
            unlockAchievement('fire_lit');
            unlitWoodpileItem.style.display = 'none';
            matchItem.style.display = 'none';
            flammableLiquidItem.style.display = 'none';
            mainButton.style.display = 'none';
            swimButton.style.display = 'block'; // Show swim button in resource tab
        }

        // Function to handle switching between screens
        function switchScreen(screen) {
            const allScreens = [gameScreen, resourceGatheringScreen, craftingScreen, achievementsScreen, progressionScreen, workersScreen, diveScreen, deepDiveScreen, commandsScreen, saveLoadScreen, statisticsScreen, bestiaryScreen, nurseryScreen, merchantScreen];
            const allTabs = [gameTab, resourceTab, craftingTab, achievementsTab, workersTab, diveTab, deepDiveTab, nurseryTab, merchantTab];

            allScreens.forEach(s => s.style.display = 'none');
            allTabs.forEach(t => t.classList.remove('active'));
            speedrunTimer.style.display = 'none';
            document.querySelector('.owner-credit').style.display = 'none';

            if (screen === 'game') {
                gameScreen.style.display = 'flex';
                gameTab.classList.add('active');
                inventoryPanel.style.display = 'flex';
                speedrunTimer.style.display = 'block';
                document.querySelector('.owner-credit').style.display = 'flex';
            } else if (screen === 'resource') {
                resourceGatheringScreen.style.display = 'flex';
                resourceTab.classList.add('active');
            } else if (screen === 'crafting') {
                craftingScreen.style.display = 'flex';
                craftingTab.classList.add('active');
                craftingIndicator.style.display = 'none';
            } else if (screen === 'achievements') {
                achievementsScreen.style.display = 'flex';
                achievementsTab.classList.add('active');
                updateAchievementScreen();
            } else if (screen === 'progression') {
                progressionScreen.style.display = 'flex';
                if (isSection2) section2.classList.remove('locked');
                else section2.classList.add('locked');
                 if (isSection3) section3.classList.remove('locked');
                else section3.classList.add('locked');
            } else if (screen === 'workers') {
                workersScreen.style.display = 'flex';
                workersTab.classList.add('active');
                workersIndicator.style.display = 'none';
                updateWorkerStats();
            } else if (screen === 'dive') {
                diveScreen.style.display = 'flex';
                diveTab.classList.add('active');
                diveIndicator.style.display = 'none';
                updateDivePrepUI();
            } else if (screen === 'deep-dive') {
                deepDiveScreen.style.display = 'flex';
                deepDiveTab.classList.add('active');
                updateDeepDivePrepUI();
            } else if (screen === 'nursery') {
                nurseryScreen.style.display = 'flex';
                nurseryTab.classList.add('active');
                updateNurseryUI();
            } else if (screen === 'merchant') {
                merchantScreen.style.display = 'flex';
                merchantTab.classList.add('active');
                updateMerchantUI();
            } else if (screen === 'commands') {
                commandsScreen.style.display = 'flex';
            } else if (screen === 'save') {
                saveLoadScreen.style.display = 'flex';
                prepareSaveData();
            } else if (screen === 'statistics') {
                statisticsScreen.style.display = 'flex';
                updateStatisticsScreen();
            } else if (screen === 'bestiary') {
                bestiaryScreen.style.display = 'flex';
                updateBestiaryScreen();
            }
        }
        
        // Function to handle the knife click
        function openKnifeModal() {
            showModal(() => cutRopeModal.style.display = 'flex');
        }

        // Function to handle the "Cut the Rope" button click
        function cutTheRope() {
            cutRopeModal.style.display = 'none';
            addNarrativeLine("You saw through the rope. The fibers snap, and you are free.");
            unlockAchievement('getting_free');
            inventoryPanel.style.display = 'none';
            knifeItem.style.display = 'none';
            mainButton.style.display = 'block';
            buttonText.textContent = 'Search';
            isFree = true;
            searchCount = 0;
        }
        
        // Function to handle music play/pause
        function toggleMusic() {
            if (backgroundMusic.paused) {
                backgroundMusic.play().catch(e => console.log("Music play failed:", e));
                isMusicPlaying = true;
                musicControlBtn.textContent = 'Music: On';
            } else {
                backgroundMusic.pause();
                isMusicPlaying = false;
                musicControlBtn.textContent = 'Music: Off';
            }
        }
        
        // --- Solo Event Logic ---
        function loseRandomItem() {
            const availableItems = [];
            if (woodCount > 0) availableItems.push('Wood');
            if (stoneCount > 0) availableItems.push('Stone');
            if (fabricCount > 0) availableItems.push('Fabric');
            if (ropeCount > 0) availableItems.push('Rope');
            if (metallicScrapCount > 0) availableItems.push('Metallic Scrap');
            
            if (availableItems.length === 0) return "Nothing was lost.";
            const itemToLose = availableItems[Math.floor(Math.random() * availableItems.length)];
            
            addItemToInventory(itemToLose, -1);
            return `A piece of ${itemToLose.toLowerCase()} was knocked overboard.`;
        }
        
        function applyTemporaryReduction(type, reduction) {
            if (isCooldownReduced) return;
            isCooldownReduced = true;
            const originalSwim = initialSwimCooldown;
            const originalGather = initialGatherCooldown;
            if (type === 'all' || type === 'swim') swimCooldownDuration *= reduction;
            if (type === 'all' || type === 'gather') gatherCooldownDuration *= reduction;
            
            setTimeout(() => {
                if (!isGamePaused) {
                    swimCooldownDuration = originalSwim;
                    gatherCooldownDuration = originalGather;
                    isCooldownReduced = false;
                    addNarrativeLine("Your cooldowns have returned to normal.");
                }
            }, cooldownReductionDuration);
        }

        const soloEvents = [
            { type: 'good', title: "A Curious Bottle", description: "You found a bottle with a strange, shimmering liquid inside.", effect: () => { applyTemporaryReduction('swim', 0.5); addNarrativeLine("Swim cooldown halved for 2 minutes!"); } },
            { type: 'good', title: "Plankton Bloom", description: "Luminous plankton makes it easier to find things.", effect: () => { applyTemporaryReduction('gather', 0.7); addNarrativeLine("Gather cooldown reduced for 2 minutes!"); } },
            { type: 'good', title: "Lost Fishing Net", description: "You find a discarded fishing net containing two pieces of Rope.", effect: () => addItemToInventory('Rope', 2) },
            { type: 'good', title: "A Swarm of Fish", description: "A school of fish passes by, leaving a sharp stone on your deck.", effect: () => addItemToInventory('Stone', 1) },
            { type: 'good', title: "Drifting Debris", description: "A large piece of driftwood floats by with metallic scrap embedded in it.", effect: () => addItemToInventory('Metallic Scrap', 2) },
            { type: 'good', title: "A Sudden Gust of Wind", description: "An unexpected gust of wind gives you a burst of energy.", effect: () => { applyTemporaryReduction('all', 0.9); addNarrativeLine("All cooldowns reduced by 10% for 2 minutes!"); } },
            { type: 'good', title: "A Calm Moment", description: "The waves and wind fall completely still. Your mind sharpens.", effect: () => { startCooldown(swimButton, swimCooldownBar, 0); startCooldown(gatherWoodButton, gatherCooldownBar, 0); addNarrativeLine("Swim and gather cooldowns have been reset!"); } },
            { type: 'bad', title: "Unexpected Scrape", description: "A jagged piece of debris scrapes the boat. A random item falls from the deck.", effect: () => addNarrativeLine(loseRandomItem()) },
            { type: 'bad', title: "A Corrupted Pouch", description: "You spot a pouch, but a rogue wave hits. The pouch is lost, and so is a random item.", effect: () => addNarrativeLine(loseRandomItem()) },
            { type: 'bad', title: "Loose Gear", description: "While adjusting your gear, a loose clasp snaps open. A random item slips into the water.", effect: () => addNarrativeLine(loseRandomItem()) }
        ];

        function triggerSoloEvent() {
            const randomEvent = soloEvents[Math.floor(Math.random() * soloEvents.length)];
            const modalFn = () => {
                soloEventModalContent.innerHTML = `
                    <p><b>Solo Event:</b> ${randomEvent.title}</p>
                    <p>${randomEvent.description}</p>
                    <button id="solo-event-continue-button" class="modal-button">Continue</button>
                `;
                document.getElementById('solo-event-continue-button').addEventListener('click', () => {
                    randomEvent.effect();
                    soloEventModal.style.display = 'none';
                    startSoloEventTimer();
                }, { once: true });
                soloEventModal.style.display = 'flex';
            };
            showModal(modalFn);
        }
        
        function checkAndTriggerSoloEvent() {
            if (isPlayerSwimming || isPlayerDiving || isPlayerDeepDiving) {
                eventQueue.push(soloEvents[Math.floor(Math.random() * soloEvents.length)]);
            } else {
                triggerSoloEvent();
            }
        }
        
        function processQueuedEvents() {
            if (eventQueue.length > 0) {
                const queuedEvent = eventQueue.shift();
                const modalFn = () => {
                    soloEventModalContent.innerHTML = `
                        <p><b>Solo Event:</b> ${queuedEvent.title}</p>
                        <p>${queuedEvent.description}</p>
                        <button id="solo-event-continue-button" class="modal-button">Continue</button>
                    `;
                    document.getElementById('solo-event-continue-button').addEventListener('click', () => {
                        queuedEvent.effect();
                        soloEventModal.style.display = 'none';
                        startSoloEventTimer();
                    }, { once: true });
                    soloEventModal.style.display = 'flex';
                };
                showModal(modalFn);
            }
        }
        
        function startSoloEventTimer() {
            if (soloEventTimer) clearTimeout(soloEventTimer);
            const randomTime = Math.random() * (420000 - 180000) + 180000;
            soloEventTimer = setTimeout(() => { if (!isGamePaused) checkAndTriggerSoloEvent(); }, randomTime);
        }
        
        function startNarrativeTimer() {
            if (narrativeTimer) clearTimeout(narrativeTimer);
            const randomTime = Math.random() * (120000 - 60000) + 60000;
            narrativeTimer = setTimeout(() => {
                if (!isGamePaused) {
                    const randomLine = section2NarrativeLines[Math.floor(Math.random() * section2NarrativeLines.length)];
                    addNarrativeLine(randomLine);
                    startNarrativeTimer();
                }
            }, randomTime);
        }

        // --- Swim Logic ---
        function swim() {
            if (sheltersBuilt > totalWorkers) {
                addNarrativeLine("You are busy building a shelter and cannot swim right now.");
                return;
            }
            // Swim lock for reinforced shelters
            if (sheltersBuilt === 5 && reinforcedSheltersBuilt > 0 && (totalWorkers - 5) / 2 < reinforcedSheltersBuilt) {
                 addNarrativeLine("You are busy building a reinforced shelter and cannot swim right now.");
                return;
            }
            if (swimButton.classList.contains('disabled')) return;

            isPlayerSwimming = true;
            gameScreen.style.display = 'none';
            resourceGatheringScreen.style.display = 'none';
            inventoryPanel.style.display = 'none';
            swimLoadingScreen.style.display = 'flex';
            loadingBar.style.width = '0%';
            
            let startTime = null;
            function updateLoadingBar(timestamp) {
                if (isGamePaused) {
                    startTime = null; // Reset start time when paused
                    requestAnimationFrame(updateLoadingBar);
                    return;
                }
                if (!startTime) startTime = timestamp;
                const elapsed = timestamp - startTime;
                const progress = (elapsed / swimLoadingDuration) * 100;
                loadingBar.style.width = progress + '%';
                if (progress < 100) requestAnimationFrame(updateLoadingBar);
                else findMaterial();
            }
            requestAnimationFrame(updateLoadingBar);

            addNarrativeLine("You dive into the cold, dark water...");
            unlockAchievement('first_swim');
        }
        
        function findMaterial() {
            isPlayerSwimming = false;
            const materials = ['Wood', 'Stone', 'Fabric', 'Rope', 'Metallic Scrap'];
            const reward = materials[Math.floor(Math.random() * materials.length)];
            const quantity = Math.floor(Math.random() * 5) + 1;
            
            swimLoadingScreen.style.display = 'none';
            const modalFn = () => {
                swimResultModal.style.display = 'flex';
                swimResultModalContent.innerHTML = `
                    <p>You found <b>${quantity} ${reward}</b>!</p>
                    <button id="collect-and-return-button" class="modal-button">Collect and return</button>
                `;
                
                document.getElementById('collect-and-return-button').addEventListener('click', () => {
                    gameContainerWrapper.classList.add('camera-shake');
                    setTimeout(() => gameContainerWrapper.classList.remove('camera-shake'), 500);

                    addItemToInventory(reward, quantity);
                    swimResultModal.style.display = 'none';
                    
                    if (!isSection2) {
                        isSection2 = true;
                        unlockAchievement('section_2_beginning');
                        addNarrativeLine("You've entered a new phase. Keep an eye out for new events...");
                        startSoloEventTimer();
                        startNarrativeTimer();
                        craftingIndicator.style.display = 'block';
                        updateShelterRecipe();
                        addCraftingRecipe(`Simple Diving Gear`, `5 Fabric, 2 Stone, 1 Metallic, 1 Rope`, `craft-diving-gear`, `diving-gear-recipe`, craftDivingGear);
                         addCraftingRecipe('Dive Bag', '10 Fabric, 5 Rope', 'craft-dive-bag', 'dive-bag-recipe', craftDiveBag, 'crafting-recipes-container', 'Increases dive inventory by 4 slots.');
                    }

                    startCooldown(swimButton, swimCooldownBar, swimCooldownDuration);
                    addNarrativeLine(`You collected ${quantity} ${reward}.`);
                    switchScreen('game');
                    processQueuedEvents();
                    processPopupQueue();
                }, { once: true });
            };
            showModal(modalFn);
        }
        
        // --- Worker Logic ---
        function updateWorkerStats() {
            idleWorkers = totalWorkers - woodGatherers - swimmers;
            totalWorkersCountSpan.textContent = totalWorkers;
            woodGatherersCountSpan.textContent = woodGatherers;
            swimmersCountSpan.textContent = swimmers;
            idleWorkersCountSpan.textContent = idleWorkers;
        }

        function assignWorker(task) {
            if (idleWorkers > 0) {
                if (task === 'wood') {
                    woodGatherers++;
                    const newInterval = setInterval(() => {
                        if (!isGamePaused) {
                            woodCount++;
                            gameStats.totalWoodCollected++;
                            woodCountSpan.textContent = woodCount;
                            woodItem.style.display = 'flex';
                            addNarrativeLine("A worker gathered a piece of wood.");
                            unlockAchievement('helpful_hands');
                        }
                    }, workerWoodInterval);
                    woodWorkerIntervals.push(newInterval);
                } else if (task === 'swim') {
                    swimmers++;
                    const newInterval = setInterval(() => {
                        if (!isGamePaused) {
                            const materials = ['Wood', 'Stone', 'Fabric', 'Rope', 'Metallic Scrap'];
                            const item = materials[Math.floor(Math.random() * materials.length)];
                            addItemToInventory(item, 1);
                            addNarrativeLine(`A worker found a piece of ${item}.`);
                            unlockAchievement('helpful_hands');
                        }
                    }, workerSwimInterval);
                    swimWorkerIntervals.push(newInterval);
                }
                updateWorkerStats();
            } else {
                addNarrativeLine("You have no idle workers to assign.");
            }
        }
        
        function recallWorker(task) {
            if (task === 'wood' && woodGatherers > 0) {
                woodGatherers--;
                clearInterval(woodWorkerIntervals.pop());
                addNarrativeLine("A wood gatherer is now idle.");
            } else if (task === 'swim' && swimmers > 0) {
                swimmers--;
                clearInterval(swimWorkerIntervals.pop());
                addNarrativeLine("A swimmer is now idle.");
            } else {
                addNarrativeLine(`You have no workers assigned to this task.`);
            }
            updateWorkerStats();
        }

        function addItemToInventory(item, quantity) {
            switch(item) {
                case 'Wood': woodCount += quantity; gameStats.totalWoodCollected += quantity > 0 ? quantity : 0; break;
                case 'Stone': stoneCount += quantity; gameStats.totalStoneCollected += quantity > 0 ? quantity : 0; break;
                case 'Fabric': fabricCount += quantity; gameStats.totalFabricCollected += quantity > 0 ? quantity : 0; break;
                case 'Rope': ropeCount += quantity; gameStats.totalRopeCollected += quantity > 0 ? quantity : 0; break;
                case 'Metallic Scrap': metallicScrapCount += quantity; gameStats.totalMetallicScrapCollected += quantity > 0 ? quantity : 0; break;
                case 'Pearl': pearlCount += quantity; break;
                case 'Harpoon Gun': hasHarpoonGun = true; break;
                case 'Harpoon Ammo': harpoonAmmoCount += quantity; break;
                case 'Bandage': bandageCount += quantity; break;
                case 'Screws': screwsCount += quantity; break;
                case 'Tungsten': tungstenCount += quantity; break;
                case 'Kevlar': kevlarCount += quantity; break;
                case 'Reinforced Stone': reinforcedStoneCount += quantity; break;
                case 'Reinforced Wood': reinforcedWoodCount += quantity; break;
                // Deep Dive Items
                case 'Coral Fragments': coralFragmentsCount += quantity; break;
                case 'Hydrothermal Sludge': hydrothermalSludgeCount += quantity; break;
                case 'Bio-Luminescence Sac': bioluminescenceSacCount += quantity; break;
                case 'Electroluminescent Filament': electroluminescentFilamentCount += quantity; break;
                case 'Chitinous Plating': chitinousPlatingCount += quantity; break;
                case 'Nautilus Shell': nautilusShellCount += quantity; break;
                case 'Ethereal Viscera': etherealVisceraCount += quantity; break;
                case 'Hydrothermal Vent Mineral': hydrothermalVentMineralCount += quantity; break;
                case 'Black Ice Core': blackIceCoreCount += quantity; break;
                case 'Giant Squid Beak': giantSquidBeakCount += quantity; break;
                case 'Abyssal Pearl': abyssalPearlCount += quantity; break;
                case 'Mythic Tooth': mythicToothCount += quantity; break;
                case 'Bone Fragment': boneFragmentCount += quantity; break;
            }
            updateAllResourceCounts();
        }
        
        // --- Diving Logic ---
        const enemies = {
            moray: { id: 'moray', name: 'Moray Eel', health: 6, minDmg: 1, maxDmg: 2, cooldown: 1500, attackName: 'Snag' },
            hatchetfish: { id: 'hatchetfish', name: 'Hatchetfish', health: 5, minDmg: 2, maxDmg: 3, cooldown: 1500, attackName: 'Chomp' },
            stargazer: { id: 'stargazer', name: 'Stargazer', health: 8, minDmg: 3, maxDmg: 3, cooldown: 2300, attackName: 'Charged Bite' },
            catshark: { id: 'catshark', name: 'Deep-Sea Catshark', health: 15, minDmg: 1, maxDmg: 1, cooldown: 1500, attackName: 'Bite' },
            spider: { id: 'spider', name: 'Colossal Sea Spider', health: 15, minDmg: 2, maxDmg: 3, cooldown: 1000, attackName: 'Stab', special: 'whip' },
            viperfish: { id: 'viperfish', name: 'Viperfish', health: 20, minDmg: 1, maxDmg: 1, cooldown: 1000, attackName: 'Snag' },
            gulpereel: { id: 'gulpereel', name: 'Gulper Eel', health: 20, minDmg: 2, maxDmg: 3, cooldown: 2000, attackName: 'Bite' },
            whalefish: { id: 'whalefish', name: 'Whalefish', health: 30, minDmg: 1, maxDmg: 1, cooldown: 500, attackName: 'Ram', special: 'headbutt' }
        };
        
        const landmarks = {
            common: [
                { name: 'Kelp Forest', lootTable: ['Metallic Scrap', 'Fabric', 'Rope'] },
                { name: 'Coral Reef', lootTable: ['Stone', 'Fabric'] },
                { name: 'Dying Coral Reef', lootTable: ['Stone', 'Metallic Scrap'] }
            ],
            rare: [
                { name: 'Sunken Battleship', lootTable: [] },
                { name: 'Sunken Yacht', lootTable: ['Fabric', 'Rope', 'Pearl'] },
                { name: 'Sunken Repair Ship', lootTable: ['Metallic Scrap', 'Screws'] },
                { name: 'Abandoned Coral Reef', lootTable: [] }
            ]
        };

        const merchantEggs = [
            { type: 'common', name: 'Common Egg', cost: 100, art: '?' },
            { type: 'complex', name: 'Complex Egg', cost: 500, art: '?' },
            { type: 'elite', name: 'Elite Egg', cost: 1000, art: '?' },
            { type: 'apocryphal', name: 'Apocryphal Egg', cost: 2000, art: '?' },
            { type: 'secret', name: 'Secret Egg', cost: 5000, art: '?' }
        ];

        function craftDivingGear() {
            if (isFreeCrafting || (fabricCount >= 5 && stoneCount >= 2 && metallicScrapCount >= 1 && ropeCount >= 1)) {
                if (!isFreeCrafting) {
                    fabricCount -= 5; stoneCount -= 2; metallicScrapCount -= 1; ropeCount -= 1;
                    updateAllResourceCounts();
                }
                hasDivingGear = true;
                divingGearItem.style.display = 'flex';
                document.getElementById('diving-gear-recipe').style.display = 'none';
                addNarrativeLine("You've crafted Simple Diving Gear. A 'Dive' tab is now available.");
                unlockAchievement('deep_diver');
                diveTab.style.display = 'block';
                diveIndicator.style.display = 'block';
                addCraftingRecipe('Spear', '2 Wood, 1 Stone, 1 Metallic, 1 Rope', 'craft-spear', 'spear-recipe', craftSpear);
                addCraftingRecipe('Better Air Tank', '5 Stone, 1 Rope, 2 Fabric', 'craft-air-tank', 'air-tank-recipe', craftBetterAirTank);
                addCraftingRecipe('Bandage', '2 Fabric, 1 Wood', 'craft-bandage', 'bandage-recipe', craftBandage);
                craftingIndicator.style.display = 'block';
            } else {
                addNarrativeLine("You don't have enough materials.");
            }
        }
        
        function craftDiveBag() {
            if (isFreeCrafting || (fabricCount >= 10 && ropeCount >= 5)) {
                if (!isFreeCrafting) {
                    fabricCount -= 10; ropeCount -= 5;
                    updateAllResourceCounts();
                }
                hasDiveBag = true;
                diveBagItem.style.display = 'flex';
                DIVE_INV_SLOTS = 8;
                document.getElementById('dive-bag-recipe').style.display = 'none';
                addNarrativeLine("You've crafted a Dive Bag, increasing your dive inventory space.");
            } else {
                addNarrativeLine("You don't have enough materials.");
            }
        }
        
        function craftFurnace() {
            if (isFreeCrafting || (metallicScrapCount >= 10 && tungstenCount >= 2 && woodCount >= 50)) {
                if (!isFreeCrafting) {
                    metallicScrapCount -= 10; tungstenCount -= 2; woodCount -= 50;
                    updateAllResourceCounts();
                }
                hasFurnace = true;
                furnaceItem.style.display = 'flex';
                document.getElementById('furnace-recipe').style.display = 'none';
                addNarrativeLine("You've crafted a Furnace. You can now craft screws.");
                addCraftingRecipe('Screws', '1 Metallic Scrap, 1 Stone', 'craft-screws', 'screws-recipe', craftScrews, 'crafting-recipes-container', 'Essential for advanced construction.');
            } else {
                addNarrativeLine("You don't have enough materials.");
            }
        }
        
        function craftScrews() {
            if (isFreeCrafting || (metallicScrapCount >= 1 && stoneCount >= 1)) {
                if (!isFreeCrafting) {
                    metallicScrapCount -= 1; stoneCount -= 1;
                    updateAllResourceCounts();
                }
                addItemToInventory('Screws', 1);
                addNarrativeLine("You crafted a screw.");
            } else {
                addNarrativeLine("You don't have enough materials.");
            }
        }
        
        function craftUnderwaterHabitat() {
            if (habitatsBuilt >= MAX_HABITATS) {
                addNarrativeLine("You have built the maximum number of habitats.");
                return;
            }
            if (isFreeCrafting || (tungstenCount >= 3 && metallicScrapCount >= 45 && screwsCount >= 100)) {
                if (!isFreeCrafting) {
                    tungstenCount -= 3; metallicScrapCount -= 45; screwsCount -= 100;
                    updateAllResourceCounts();
                }
                habitatsBuilt++;
                addNarrativeLine(`You've constructed an Underwater Habitat module (${habitatsBuilt}/${MAX_HABITATS}).`);
                if (habitatsBuilt >= MAX_HABITATS) {
                    document.getElementById('habitat-recipe').style.display = 'none';
                }
            } else {
                addNarrativeLine("You don't have enough materials.");
            }
        }


        function craftSpear() {
            if (hasSpear) {
                addNarrativeLine("You already have a spear.");
                return;
            }
            if (isFreeCrafting || (woodCount >= 2 && stoneCount >= 1 && metallicScrapCount >= 1 && ropeCount >= 1)) {
                if (!isFreeCrafting) {
                    woodCount -= 2; stoneCount -= 1; metallicScrapCount -= 1; ropeCount -= 1;
                    updateAllResourceCounts();
                }
                hasSpear = true;
                spearItem.style.display = 'flex';
                addNarrativeLine("You've crafted a sturdy Spear.");
            } else { addNarrativeLine("You lack the materials."); }
        }

        function craftBetterAirTank() {
            if (isFreeCrafting || (stoneCount >= 5 && ropeCount >= 1 && fabricCount >= 2)) {
                if (!isFreeCrafting) {
                    stoneCount -= 5; ropeCount -= 1; fabricCount -= 2;
                    updateAllResourceCounts();
                }
                hasBetterAirTank = true;
                airTankItem.style.display = 'flex';
                addNarrativeLine("You've crafted a Better Air Tank.");
            } else { addNarrativeLine("You can't craft this without the right materials."); }
        }
        
        function craftFishingNet() {
            if (isFreeCrafting || ropeCount >= 20) {
                if (!isFreeCrafting) {
                    ropeCount -= 20;
                    updateAllResourceCounts();
                }
                hasFishingNet = true;
                fishingNetItem.style.display = 'flex';
                addNarrativeLine("You've crafted a Fishing Net.");
            } else { addNarrativeLine("You need 20 rope."); }
        }
        
        function craftBandage() {
            if (isFreeCrafting || (fabricCount >= 2 && woodCount >= 1)) {
                if (!isFreeCrafting) {
                    fabricCount -= 2; woodCount -= 1;
                    updateAllResourceCounts();
                }
                addItemToInventory('Bandage', 1);
                addNarrativeLine("You've crafted a simple bandage.");
            } else { addNarrativeLine("You need 2 Fabric and 1 Wood."); }
        }
        
        function craftHarpoonGun() {
            if (isFreeCrafting || (tungstenCount >= 5 && kevlarCount >= 1 && reinforcedStoneCount >= 2 && ropeCount >= 10)) {
                if (!isFreeCrafting) {
                    tungstenCount -= 5; kevlarCount -= 1; reinforcedStoneCount -= 2; ropeCount -= 10;
                    updateAllResourceCounts();
                }
                hasHarpoonGun = true;
                harpoonGunItem.style.display = 'flex';
                document.getElementById('harpoon-gun-recipe').style.display = 'none';
                addNarrativeLine("You've crafted a powerful Harpoon Gun.");
            } else { addNarrativeLine("You lack the materials for the Harpoon Gun."); }
        }

        function craftHarpoonAmmo() {
            if (isFreeCrafting || (reinforcedStoneCount >= 1 && metallicScrapCount >= 1)) {
                if (!isFreeCrafting) {
                    reinforcedStoneCount -= 1; metallicScrapCount -= 1;
                    updateAllResourceCounts();
                }
                addItemToInventory('Harpoon Ammo', 2);
                addNarrativeLine("You crafted 2 Harpoon Ammo.");
            } else { addNarrativeLine("You need 1 Reinforced Stone and 1 Metallic Scrap."); }
        }
        
        function craftTungsten() {
            if (isFreeCrafting || metallicScrapCount >= 50) {
                if (!isFreeCrafting) { metallicScrapCount -= 50; updateAllResourceCounts(); }
                addItemToInventory('Tungsten', 1);
                addNarrativeLine("You smelted scrap into a dense Tungsten ingot.");
            } else { addNarrativeLine("Not enough Metallic Scrap."); }
        }
        function craftKevlar() {
            if (isFreeCrafting || fabricCount >= 100) {
                if (!isFreeCrafting) { fabricCount -= 100; updateAllResourceCounts(); }
                addItemToInventory('Kevlar', 1);
                addNarrativeLine("You wove fabric into a sheet of durable Kevlar.");
                // Unlock Complex Diving Gear recipe on first craft
                if (!document.getElementById('complex-diving-gear-recipe')) {
                    addCraftingRecipe('Complex Diving Gear', '2 Kevlar, 5 Tungsten, 10 R. Stone', 'craft-complex-gear', 'complex-diving-gear-recipe', craftComplexDivingGear, 'purity-recipes-container', 'Allows for Deep Dives.');
                    addCraftingRecipe('Kevlar Armour', '5 Kevlar, 10 Rope', 'craft-kevlar-armour', 'kevlar-armour-recipe', craftKevlarArmour, 'purity-recipes-container', 'Increases max health to 25.');
                    craftingIndicator.style.display = 'block';
                }
            } else { addNarrativeLine("Not enough Fabric."); }
        }
        
        function craftKevlarArmour() {
            if (isFreeCrafting || (kevlarCount >= 5 && ropeCount >= 10)) {
                if (!isFreeCrafting) {
                    kevlarCount -= 5;
                    ropeCount -= 10;
                    updateAllResourceCounts();
                }
                hasKevlarArmour = true;
                kevlarArmourItem.style.display = 'flex';
                DEFAULT_MAX_PLAYER_HEALTH = 25;
                MAX_PLAYER_HEALTH = hasGodHealth ? 500 : DEFAULT_MAX_PLAYER_HEALTH;
                playerHealth = MAX_PLAYER_HEALTH; // Heal to new max
                document.getElementById('kevlar-armour-recipe').style.display = 'none';
                addNarrativeLine("You've crafted Kevlar Armour, increasing your resilience.");
            } else {
                addNarrativeLine("You don't have enough materials for Kevlar Armour.");
            }
        }
        
        function craftReinforcedStone() {
            if (isFreeCrafting || stoneCount >= 50) {
                if (!isFreeCrafting) { stoneCount -= 50; updateAllResourceCounts(); }
                addItemToInventory('Reinforced Stone', 1);
                addNarrativeLine("You compressed stones into a block of Reinforced Stone.");
            } else { addNarrativeLine("Not enough Stone."); }
        }
        function craftReinforcedWood() {
            if (isFreeCrafting || woodCount >= 50) {
                if (!isFreeCrafting) { woodCount -= 50; updateAllResourceCounts(); }
                addItemToInventory('Reinforced Wood', 1);
                addNarrativeLine("You treated and bound wood, making it unnaturally strong.");
            } else { addNarrativeLine("Not enough Wood."); }
        }
        
        function craftComplexDivingGear() {
            if (isFreeCrafting || (kevlarCount >= 2 && tungstenCount >= 5 && reinforcedStoneCount >= 10)) {
                if (!isFreeCrafting) {
                    kevlarCount -= 2;
                    tungstenCount -= 5;
                    reinforcedStoneCount -= 10;
                    updateAllResourceCounts();
                }
                hasComplexDivingGear = true;
                complexDivingGearItem.style.display = 'flex';
                document.getElementById('complex-diving-gear-recipe').style.display = 'none';
                addNarrativeLine("You've crafted Complex Diving Gear. A 'Deep Dive' tab is now available.");
                deepDiveTab.style.display = 'block';
                unlockAchievement('abyss_gazer');
            } else {
                addNarrativeLine("You don't have enough materials for the Complex Diving Gear.");
            }
        }


        function updateDivePrepUI() {
            toolSelectionContainer.innerHTML = '';
            if (hasSpear) toolSelectionContainer.innerHTML += `<div><input type="checkbox" id="select-spear" value="spear"><label for="select-spear"> Bring Spear</label></div>`;
            if (hasHarpoonGun) toolSelectionContainer.innerHTML += `<div><input type="checkbox" id="select-harpoon" value="harpoon"><label for="select-harpoon"> Bring Harpoon Gun (${harpoonAmmoCount} ammo)</label></div>`;
            if (hasBetterAirTank) toolSelectionContainer.innerHTML += `<div><input type="checkbox" id="select-air-tank" value="air-tank"><label for="select-air-tank"> Bring Better Air Tank (+20 Moves)</label></div>`;
            if (hasFishingNet) toolSelectionContainer.innerHTML += `<div><input type="checkbox" id="select-net" value="net"><label for="select-net"> Bring Fishing Net</label></div>`;
            if (bandageCount > 0) toolSelectionContainer.innerHTML += `<div><label for="select-bandages">Bring Bandages: </label><input type="number" id="select-bandages" value="0" min="0" max="${bandageCount}" style="width: 50px;"><span> (Max: ${bandageCount})</span></div>`;
            if (toolSelectionContainer.innerHTML === '') toolSelectionContainer.innerHTML = '<p>You have no special tools to bring.</p>';
        }

        function generateMinigameMap() {
            minigameMap = Array(MAP_SIZE).fill(null).map(() => Array(MAP_SIZE).fill({ type: 'path' }));
            const center = Math.floor(MAP_SIZE / 2);
            
            const totalLandmarks = 20;
            for (let i = 0; i < totalLandmarks; i++) {
                const x = Math.floor(Math.random() * MAP_SIZE);
                const y = Math.floor(Math.random() * MAP_SIZE);
                const distFromCenter = Math.sqrt(Math.pow(x - center, 2) + Math.pow(y - center, 2));
                let landmarkType = (distFromCenter > MAP_SIZE * 0.3 && Math.random() < 0.3) ? landmarks.rare[Math.floor(Math.random() * landmarks.rare.length)] : landmarks.common[Math.floor(Math.random() * landmarks.common.length)];
                if (x !== center || y !== center) minigameMap[y][x] = { type: 'landmark', details: landmarkType };
            }

            minigameMap[center][center] = { type: 'boat' };
            playerPosition = { x: center, y: center };
        }

        function renderMinigame() {
            let gridHTML = '';
            for (let y = 0; y < MAP_SIZE; y++) {
                let rowHTML = '';
                for (let x = 0; x < MAP_SIZE; x++) {
                    const isSafe = safePaths.some(p => p.x === x && p.y === y);
                    const isHabitat = placedHabitats.some(h => h.x === x && h.y === y);
                    const tile = minigameMap[y][x];
                    if (x === playerPosition.x && y === playerPosition.y) {
                         let playerIcon = '';
                        if (equippedPets[0]) playerIcon += `<span class="pet">${equippedPets[0].icon}</span>`;
                        playerIcon += '<span class="player">@</span>';
                        if (equippedPets[1]) playerIcon += `<span class="pet">${equippedPets[1].icon}</span>`;
                        rowHTML += playerIcon;
                    }
                    else if (isHabitat) rowHTML += '<span class="habitat">H</span>';
                    else if (tile.type === 'boat') rowHTML += '<span class="boat">$</span>';
                    else if (tile.type === 'landmark') rowHTML += `<span class="landmark" data-name="${tile.details.name}">#<span class="tooltip-text">${tile.details.name}</span></span>`;
                    else if (isSafe) rowHTML += '<span class="safe-path">.</span>';
                    else rowHTML += '*';
                }
                gridHTML += rowHTML + '\n';
            }
            minigameGrid.innerHTML = gridHTML;
            movesLeftSpan.textContent = hasInfiniteMoves ? '∞' : diveMoves;
            updateHealthBars();
            diveBandageCountSpan.textContent = diveBandages;
            useBandageButton.style.display = diveBandages > 0 ? 'inline-block' : 'none';
            buildHabitatButton.style.display = habitatsBuilt > placedHabitats.length ? 'inline-block' : 'none';
            updateDiveInventoryDisplay();
        }

        function beginDive() {
            if (sheltersBuilt > totalWorkers) {
                addNarrativeLine("You are busy building a shelter and cannot dive right now.");
                return;
            }
            if (beginDiveButton.classList.contains('disabled')) return;
            
            rareEnemyPathwayChance = 0.05; // Reset rare enemy chance
            isPlayerDiving = true;
            gameStats.totalDives++;
            divePrepArea.style.display = 'none';
            diveMinigameArea.style.display = 'flex';
            
            // Reset used status for habitats at the start of a new dive
            placedHabitats.forEach(h => h.usedThisDive = false);
            
            MAX_PLAYER_HEALTH = hasGodHealth ? 500 : (hasKevlarArmour ? 25 : 15);
            playerHealth = MAX_PLAYER_HEALTH;
            diveMoves = 20;
            diveTools = [];
            playerPath = [];

            const spearCheckbox = document.getElementById('select-spear');
            const airTankCheckbox = document.getElementById('select-air-tank');
            const netCheckbox = document.getElementById('select-net');
            const harpoonCheckbox = document.getElementById('select-harpoon');
            const bandageInput = document.getElementById('select-bandages');
            
            diveBandages = 0;
            if (bandageInput && parseInt(bandageInput.value) > 0) {
                const bandagesToTake = Math.min(bandageCount, parseInt(bandageInput.value));
                diveBandages = bandagesToTake;
                bandageCount -= bandagesToTake;
                bandageCountSpan.textContent = bandageCount;
            }

            if (airTankCheckbox && airTankCheckbox.checked) { diveMoves += 20; diveTools.push('air-tank'); }
            if (spearCheckbox && spearCheckbox.checked) { diveTools.push('spear'); }
            if (netCheckbox && netCheckbox.checked) { diveTools.push('net'); }
            if (harpoonCheckbox && harpoonCheckbox.checked) { diveTools.push('harpoon'); }

            diveInventory = [];
            generateMinigameMap(); // This regenerates landmarks every dive
            renderMinigame();
            document.addEventListener('keydown', handlePlayerMovement);
        }
        
        function handlePlayerMovement(e) {
            if (isPlayerInCombat) {
                 if (e.key === '1') playerAttack('fist');
                 else if (e.key === '2' && (diveTools.includes('spear') || deepDiveTools.includes('spear'))) playerAttack('spear');
                 else if (e.key === '3' && (diveTools.includes('net') || deepDiveTools.includes('net'))) playerTangle();
                 else if (e.key === '4' && (diveTools.includes('harpoon') || deepDiveTools.includes('harpoon'))) playerAttack('harpoon');
                 else if (e.key === '0') useBandage();
                return;
            }
            
            if (isPlayerDiving) {
                handleDiveMovement(e);
            } else if (isPlayerDeepDiving) {
                handleDeepDiveMovement(e);
            }
        }
        
        function handleDiveMovement(e) {
            let newX = playerPosition.x, newY = playerPosition.y;
            if (e.key === 'ArrowUp' || e.key.toLowerCase() === 'w') newY--;
            else if (e.key === 'ArrowDown' || e.key.toLowerCase() === 's') newY++;
            else if (e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a') newX--;
            else if (e.key === 'ArrowRight' || e.key.toLowerCase() === 'd') newX++;
            else return;

            if (newY >= 0 && newY < MAP_SIZE && newX >= 0 && newX < MAP_SIZE) {
                playerPosition = { x: newX, y: newY };
                playerPath.push({x: newX, y: newY});
                if (!hasInfiniteMoves) diveMoves--;

                const currentTile = minigameMap[newY][newX];
                const habitatAtLocation = placedHabitats.find(h => h.x === newX && h.y === newY);
                const isOnSafePath = safePaths.some(p => p.x === newX && p.y === newY);

                if (habitatAtLocation && !habitatAtLocation.usedThisDive) {
                    useHabitat(habitatAtLocation);
                }
                else if (currentTile.type === 'path' && !noEnemies && !isOnSafePath) {
                    // Check for rare enemies first
                    if (Math.random() < rareEnemyPathwayChance) {
                        const rareEnemies = [enemies.whalefish, enemies.spider];
                        startCombat(rareEnemies[Math.floor(Math.random() * rareEnemies.length)]);
                        rareEnemyPathwayChance = 0.05; // Reset chance
                        return; // Stop further checks
                    } 
                    // Then check for common enemies
                    else if (Math.random() < 0.15) { // 15% chance for common
                        const commonEnemies = Object.values(enemies).filter(e => !['whalefish', 'spider'].includes(e.id));
                        startCombat(commonEnemies[Math.floor(Math.random() * commonEnemies.length)]);
                        return; // Stop further checks
                    }
                    // If no enemy spawned, increase rare chance
                    rareEnemyPathwayChance += 0.0045;

                } else if (currentTile.type === 'landmark') {
                    openLandmark(currentTile.details);
                    minigameMap[newY][newX] = { type: 'path' };
                } else if (currentTile.type === 'boat') {
                    endDive(true);
                    return;
                }

                renderMinigame();
                if (diveMoves <= 0 && !hasInfiniteMoves) endDive(false);
            }
        }
        
        function endDive(isSuccess) {
            document.removeEventListener('keydown', handlePlayerMovement);
            isPlayerDiving = false;
            diveMinigameArea.style.display = 'none';
            divePrepArea.style.display = 'flex';
            startCooldown(beginDiveButton, diveCooldownBar, diveCooldownDuration);
            MAX_PLAYER_HEALTH = hasKevlarArmour ? 25 : 15; // Reset health
            rareEnemyPathwayChance = 0.05; // Reset rare enemy chance
            
            bandageCount += diveBandages; // Return unused bandages
            bandageCountSpan.textContent = bandageCount;
            
            if (isSuccess) {
                addNarrativeLine("You returned to the boat safely.");
                // Merge unique path coordinates into safePaths
                const pathSet = new Set(safePaths.map(p => `${p.x},${p.y}`));
                playerPath.forEach(p => pathSet.add(`${p.x},${p.y}`));
                safePaths = Array.from(pathSet).map(s => ({ x: parseInt(s.split(',')[0]), y: parseInt(s.split(',')[1]) }));

                diveInventory.forEach(slot => {
                    if (slot) {
                        addItemToInventory(slot.item, slot.quantity);
                        addNarrativeLine(`Brought back ${slot.quantity} ${slot.item}.`);
                    }
                });
                if (!document.getElementById('fishing-net-recipe')) {
                    addCraftingRecipe('Fishing Net', '20 Rope', 'craft-fishing-net', 'fishing-net-recipe', craftFishingNet, 'crafting-recipes-container', 'Used to tangle enemies in combat.');
                    purityCraftingContainer.style.display = 'block';
                    addCraftingRecipe('Tungsten', '50 Metallic Scrap', 'craft-tungsten', 'tungsten-recipe', craftTungsten, 'purity-recipes-container');
                    addCraftingRecipe('Kevlar', '100 Fabric', 'craft-kevlar', 'kevlar-recipe', craftKevlar, 'purity-recipes-container');
                    addCraftingRecipe('Reinforced Stone', '50 Stone', 'craft-r-stone', 'r-stone-recipe', craftReinforcedStone, 'purity-recipes-container');
                    addCraftingRecipe('Reinforced Wood', '50 Wood', 'craft-r-wood', 'r-wood-recipe', craftReinforcedWood, 'purity-recipes-container');
                    craftingIndicator.style.display = 'block';
                }
            } else {
                addNarrativeLine("You passed out... You were lucky to float back, but lost everything from the dive.");
                if (hasSpear) {
                    hasSpear = false;
                    spearItem.style.display = 'none';
                    addNarrativeLine("Your spear was lost in the depths.");
                }
                 if (!hasFoundRepairShip) { // Still unlock recipes on first dive failure
                    showRepairShipRecipes();
                }
            }
            diveInventory = [];
            diveTools = [];
            
            switchScreen('game');
            processQueuedEvents();
            processPopupQueue();
        }
        
        // --- Landmark & Combat Logic ---
        function openLandmark(details, isDeepDive = false) {
             if (details.name === 'Sunken Battleship') {
                showModal(() => battleshipChoiceModal.style.display = 'flex');
                if (!hasFoundBattleship) {
                    hasFoundBattleship = true;
                    addCraftingRecipe('Harpoon Gun', '5 Tungsten, 1 Kevlar, 2 R. Stone, 10 Rope', 'craft-harpoon-gun', 'harpoon-gun-recipe', craftHarpoonGun);
                    addCraftingRecipe('Harpoon Ammo (x2)', '1 R. Stone, 1 Metallic Scrap', 'craft-harpoon-ammo', 'harpoon-ammo-recipe', craftHarpoonAmmo);
                    craftingIndicator.style.display = 'block';
                    addNarrativeLine("You found schematics for a Harpoon Gun!");
                }
                return;
            }
            
            if (details.name === 'Abandoned Coral Reef' && !hasUnlockedNursery) {
                hasUnlockedNursery = true;
                nurseryTab.style.display = 'block';
                merchantTab.style.display = 'block';
                addNarrativeLine("You've discovered a strange, vibrant coral reef. It seems to be a nursery for exotic sea life. New tabs are available.");
                const modalFn = () => {
                    const popup = document.createElement('div');
                    popup.classList.add('modal');
                    popup.style.display = 'flex';
                    popup.innerHTML = `<div class="modal-content"><p>You have unlocked the Nursery and Egg Merchant tabs!</p></div>`;
                    document.body.appendChild(popup);
                    setTimeout(() => {
                        popup.remove();
                    }, 3000); // Popup disappears after 3 seconds
                };
                showModal(modalFn);
            }


             if (details.name === 'Sunken Repair Ship' && !hasFoundRepairShip) {
                showRepairShipRecipes();
                hasFoundRepairShip = true;
                const modalFn = () => {
                    const popup = document.createElement('div');
                    popup.classList.add('modal');
                    popup.style.display = 'flex';
                    popup.innerHTML = `<div class="modal-content"><p>You've discovered new crafting recipes!</p><button class="modal-button">OK</button></div>`;
                    document.body.appendChild(popup);
                    popup.querySelector('button').addEventListener('click', () => popup.remove(), { once: true });
                };
                showModal(modalFn);
            }
            
            let currentInventory = isDeepDive ? deepDiveInventory : diveInventory;
            let tempDiveInv = JSON.parse(JSON.stringify(currentInventory)); // Create a temporary copy for this interaction
            
            const renderLandmark = (secondPhase = false) => {
                let lootHTML = `<h3>${secondPhase ? 'Further Exploration' : 'You explored the'} ${details.name}</h3>`;
                const lootOptions = [];
                const availableLoot = details.lootTable.length > 0 ? details.lootTable : ['Wood', 'Stone', 'Fabric', 'Rope', 'Metallic Scrap'];
                for (let i = 0; i < 4; i++) {
                    const item = availableLoot[Math.floor(Math.random() * availableLoot.length)];
                    const quantity = Math.floor(Math.random() * 4) + 1; // 1-4
                    lootOptions.push({ item, quantity });
                }

                lootHTML += '<p>Choose what to take:</p><div id="landmark-choices">';
                lootOptions.forEach((opt, i) => {
                    lootHTML += `<button class="modal-button choice-button" data-item="${opt.item}" data-quantity="${opt.quantity}">${opt.quantity} ${opt.item}</button>`;
                });
                lootHTML += '</div><div id="landmark-actions">';
                if (!secondPhase && !isDeepDive) lootHTML += `<button id="explore-further" class="modal-button">Explore Further</button>`;
                lootHTML += `<button id="confirm-landmark" class="modal-button">Confirm & Leave</button></div>`;
                landmarkModalContent.innerHTML = lootHTML;
                
                renderDiveInventoryPopup(tempDiveInv);
                
                document.querySelectorAll('.choice-button').forEach(btn => {
                    btn.addEventListener('click', () => {
                        const item = btn.dataset.item;
                        const quantity = parseInt(btn.dataset.quantity);
                        const leftover = addToDiveInventory(item, quantity, tempDiveInv);
                        btn.dataset.quantity = leftover;
                        btn.textContent = `${leftover} ${item}`;
                        if (leftover === 0) btn.disabled = true;
                        renderDiveInventoryPopup(tempDiveInv);
                    });
                });

                if (!secondPhase && !isDeepDive) {
                    document.getElementById('explore-further').addEventListener('click', () => {
                        diveInventory = tempDiveInv; // Save current choices before exploring
                        if (Math.random() < 0.18) { // 18% chance for rare enemy
                            landmarkModalContainer.style.display = 'none';
                            const rareEnemies = [enemies.whalefish, enemies.spider];
                            startCombat(rareEnemies[Math.floor(Math.random() * rareEnemies.length)], true);
                            rareEnemyPathwayChance = 0.05; // Reset chance
                        } else {
                            renderLandmark(true);
                        }
                    }, { once: true });
                }

                document.getElementById('confirm-landmark').addEventListener('click', () => {
                    if (isDeepDive) {
                        deepDiveInventory = tempDiveInv;
                    } else {
                        diveInventory = tempDiveInv;
                    }
                    landmarkModalContainer.style.display = 'none';
                    if (isDeepDive) renderDeepDiveMinigame(); else renderMinigame();
                }, { once: true });
            };
            
            showModal(() => {
                landmarkModalContainer.style.display = 'flex';
                renderLandmark();
            });
        }
        
        function renderDiveInventoryPopup(inventory) {
            let invHTML = '';
            for (let i = 0; i < DIVE_INV_SLOTS; i++) {
                const slot = inventory[i];
                if (slot) {
                    invHTML += `<div>${slot.item}: ${slot.quantity}/${DIVE_STACK_LIMIT} <button class="modal-button" data-slot="${i}" style="font-size: 0.7rem; padding: 2px 4px;">Drop</button></div>`;
                } else {
                    invHTML += `<div><i>- Empty -</i></div>`;
                }
            }
            diveInventoryPopupContent.innerHTML = invHTML;
            diveInventoryPopupContent.querySelectorAll('button').forEach(btn => {
                btn.addEventListener('click', () => {
                    inventory.splice(parseInt(btn.dataset.slot), 1);
                    renderDiveInventoryPopup(inventory);
                });
            });
        }

        function addToDiveInventory(item, quantity, inventory) {
            let remainingQuantity = quantity;
            // Try to add to existing stacks first
            for (const slot of inventory) {
                if (slot && slot.item === item && slot.quantity < DIVE_STACK_LIMIT) {
                    const space = DIVE_STACK_LIMIT - slot.quantity;
                    const amountToAdd = Math.min(remainingQuantity, space);
                    slot.quantity += amountToAdd;
                    remainingQuantity -= amountToAdd;
                    if (remainingQuantity === 0) return 0;
                }
            }
            // Try to add to a new slot
            while (remainingQuantity > 0 && inventory.length < DIVE_INV_SLOTS) {
                const amountToAdd = Math.min(remainingQuantity, DIVE_STACK_LIMIT);
                inventory.push({ item: item, quantity: amountToAdd });
                remainingQuantity -= amountToAdd;
            }
            return remainingQuantity;
        }

        function useBandage() {
            let currentBandages = isPlayerDeepDiving ? deepDiveBandages : diveBandages;
            
            if (isHealOnCooldown && isPlayerInCombat) addCombatLog("Bandage is on cooldown.");
            else if (currentBandages <= 0) addCombatLog("You have no bandages.");
            else if (playerHealth >= MAX_PLAYER_HEALTH) addCombatLog("You are already at full health.");
            else {
                const healedAmount = Math.min(3, MAX_PLAYER_HEALTH - playerHealth);
                playerHealth += healedAmount;
                
                if(isPlayerDeepDiving) deepDiveBandages--; else diveBandages--;
                
                if (isPlayerInCombat) {
                    addCombatLog(`You used a bandage and healed for ${healedAmount} HP.`);
                    isHealOnCooldown = true;
                    startCooldown(healButton, healCooldownBar, healCooldownDuration);
                    setTimeout(() => { isHealOnCooldown = false; }, getCooldownDuration(healCooldownDuration));
                    updateHealthBars();
                } else {
                    addNarrativeLine(`You used a bandage and healed for ${healedAmount} HP.`);
                    if(isPlayerDeepDiving) renderDeepDiveMinigame(); else renderMinigame();
                }
            }
        }
        
        function startCombat(enemyData, noLoot = false) {
            isPlayerInCombat = true;
            currentEnemy = { ...enemyData, noLoot };
            if (currentEnemy.special) {
                currentEnemy.special.lastUsed = -999; // Ensure special can be used on first turn
            }
            combatTurnCounter = 0;
            enemyHealth = currentEnemy.health;
            maxEnemyHealth = currentEnemy.health;
            
            // Add to bestiary if it's a new encounter
            const allEnemies = {...enemies, ...deepDiveEnemies};
            const enemyId = Object.keys(allEnemies).find(key => allEnemies[key].name === enemyData.name);
            if(enemyId) encounteredEnemies.add(enemyId);
            
            showModal(() => {
                combatModal.style.display = 'flex';
                enemyName.textContent = currentEnemy.name;
                updateHealthBars();
                combatLog.innerHTML = `<div>A wild ${currentEnemy.name} appears!</div>`;
                
                let currentTools = isPlayerDeepDiving ? deepDiveTools : diveTools;
                spearAttackButton.style.display = currentTools.includes('spear') ? 'inline-block' : 'none';
                tangleButton.style.display = currentTools.includes('net') ? 'inline-block' : 'none';
                harpoonAttackButton.style.display = currentTools.includes('harpoon') && harpoonAmmoCount > 0 ? 'inline-block' : 'none';
                
                enemyAttackInterval = setInterval(enemyAttack, currentEnemy.cooldown);
            });
        }
        
        function playerAttack(type) {
            if (!isPlayerInCombat || isPlayerStunned) return;
            
            if (Math.random() < 0.10) { // 10% miss chance
                addCombatLog("Your attack missed!");
                return;
            }

            let damage = 0, cooldown = 850, attackName = '';
            if (type === 'fist' && !fistAttackButton.classList.contains('disabled') && !isFistDisabled) {
                damage = Math.floor(Math.random() * 2) + 1;
                attackName = 'punched';
                startCooldown(fistAttackButton, null, cooldown);
            } else if (type === 'spear' && !spearAttackButton.classList.contains('disabled')) {
                damage = Math.floor(Math.random() * 3) + 2;
                attackName = 'shanked';
                startCooldown(spearAttackButton, null, cooldown);
            } else if (type === 'harpoon' && !harpoonAttackButton.classList.contains('disabled')) {
                if (harpoonAmmoCount > 0) {
                    damage = 5;
                    attackName = 'shot';
                    harpoonAmmoCount--;
                    startCooldown(harpoonAttackButton, null, harpoonCooldownDuration);
                    if (harpoonAmmoCount <= 0) harpoonAttackButton.style.display = 'none';
                } else {
                    addCombatLog("Out of harpoon ammo!");
                    return;
                }
            } else return;
            
            enemyHealth -= damage;
            addCombatLog(`You ${attackName} the ${currentEnemy.name} for ${damage} damage.`);
            updateHealthBars();
            if (enemyHealth <= 0) endCombat('win');
        }
        
        function playerTangle() {
            if (!isPlayerInCombat || isPlayerStunned || tangleButton.classList.contains('disabled')) return;
            
            addCombatLog(`You tangle the ${currentEnemy.name}!`);
            isEnemyStunned = true;
            clearInterval(enemyAttackInterval); // Stop enemy attacks
            startCooldown(tangleButton, tangleCooldownBar, tangleCooldownDuration);
            
            // Set a timeout to unstun the enemy
            enemyStunTimeout = setTimeout(() => {
                if (isPlayerInCombat) { // Only if combat is still ongoing
                    addCombatLog(`The ${currentEnemy.name} broke free!`);
                    isEnemyStunned = false;
                    enemyAttackInterval = setInterval(enemyAttack, currentEnemy.cooldown);
                }
                enemyStunTimeout = null;
            }, 2000);
        }
        
        function enemyAttack() {
            if (isEnemyStunned || isGamePaused) return;
            combatTurnCounter++;

            // Handle player DoT
            if (playerDoT.duration > 0) {
                playerHealth -= playerDoT.damage;
                addCombatLog(`You take ${playerDoT.damage} damage from Ethereal Envelopment.`);
                playerDoT.duration--;
                if (playerDoT.duration <= 0) addCombatLog("The ethereal effect fades.");
                updateHealthBars();
                if (playerHealth <= 0) { endCombat('loss'); return; }
            }
            
            // Check for special moves
            if (currentEnemy.special && (combatTurnCounter - currentEnemy.special.lastUsed) * currentEnemy.cooldown >= currentEnemy.special.cooldown) {
                currentEnemy.special.lastUsed = combatTurnCounter;
                handleEnemySpecial(currentEnemy.special.name);
                return;
            }
            
            if (currentEnemy.special2 && (combatTurnCounter - currentEnemy.special2.lastUsed) * currentEnemy.cooldown >= currentEnemy.special2.cooldown) {
                currentEnemy.special2.lastUsed = combatTurnCounter;
                handleEnemySpecial(currentEnemy.special2.name);
                return;
            }

            if (Math.random() < 0.2) {
                addCombatLog(`The ${currentEnemy.name}'s attack missed!`);
                return;
            }
            
            const damage = Math.floor(Math.random() * (currentEnemy.maxDmg - currentEnemy.minDmg + 1)) + currentEnemy.minDmg;
            playerHealth -= damage;
            addCombatLog(`The ${currentEnemy.name} used ${currentEnemy.attackName} for ${damage} damage.`);
            updateHealthBars();
            if (playerHealth <= 0) endCombat('loss');
        }
        
        function attemptFlee() {
            if (fleeButton.classList.contains('disabled')) return;
            startCooldown(fleeButton, null, 2000);
            if (Math.random() < 0.3) {
                addCombatLog("You successfully escaped!");
                endCombat('flee');
            } else {
                addCombatLog("You failed to escape!");
            }
        }
        
        function endCombat(result) {
            clearInterval(enemyAttackInterval);
            if (enemyStunTimeout) {
                clearTimeout(enemyStunTimeout);
                enemyStunTimeout = null;
            }
            isPlayerInCombat = false;
            playerDoT = { damage: 0, duration: 0 }; // Clear any DoT
            
            if (result === 'win') {
                const enemyType = currentEnemy.name;
                if (!gameStats.enemiesSlain[enemyType]) gameStats.enemiesSlain[enemyType] = 0;
                gameStats.enemiesSlain[enemyType]++;
                
                if (isPlayerDeepDiving && !unlockedAchievements.has('first_kill_deep')) {
                    unlockAchievement('first_kill_deep');
                }


                setTimeout(() => {
                    combatModal.style.display = 'none';
                    if (currentEnemy.noLoot) {
                        addNarrativeLine(`You defeated the ${currentEnemy.name}!`);
                        openLandmark(landmarks.common[0]); // Re-open a generic landmark
                    } else {
                        showLootPopup();
                    }
                }, 1500);
            } else if (result === 'loss') {
                setTimeout(() => {
                    combatModal.style.display = 'none';
                    addNarrativeLine("You were defeated and blacked out...");
                    if (isPlayerDeepDiving) endDeepDive(false); else endDive(false);
                }, 1500);
            } else if (result === 'flee') {
                 setTimeout(() => {
                    combatModal.style.display = 'none';
                    if (isPlayerDeepDiving) renderDeepDiveMinigame(); else renderMinigame();
                }, 1500);
            }
        }

        function showLootPopup() {
            const enemyType = currentEnemy.name;
            let droppedItem, dropQuantity;
            
            // Regular Dive Loot
            if (Object.values(enemies).some(e => e.name === enemyType)) {
                 if (['Colossal Sea Spider', 'Whalefish'].includes(enemyType)) {
                    if (Math.random() < 0.45) {
                        const purityMaterials = ['Tungsten', 'Kevlar', 'Reinforced Stone', 'Reinforced Wood'];
                        droppedItem = purityMaterials[Math.floor(Math.random() * purityMaterials.length)];
                        dropQuantity = 1; // Only drops 1
                    } else {
                        const dropTable = ['Stone', 'Fabric', 'Rope', 'Metallic Scrap'];
                        droppedItem = dropTable[Math.floor(Math.random() * dropTable.length)];
                        dropQuantity = Math.floor(Math.random() * 6) + 5;
                    }
                } else {
                    const dropTable = ['Stone', 'Fabric', 'Rope', 'Metallic Scrap'];
                    droppedItem = dropTable[Math.floor(Math.random() * dropTable.length)];
                    dropQuantity = (['Viperfish', 'Gulper Eel', 'Deep-Sea Catshark'].includes(enemyType)) ? Math.floor(Math.random() * 4) + 5 : Math.floor(Math.random() * 3) + 1;
                }
            } 
            // Deep Dive Loot
            else {
                const drop = currentEnemy.drops[Math.floor(Math.random() * currentEnemy.drops.length)];
                droppedItem = drop.item;
                dropQuantity = Math.floor(Math.random() * (drop.max - drop.min + 1)) + drop.min;
                
                // Pearl Drop Logic
                if (hasUnlockedNursery) {
                    let pearlDrop = 0;
                    const rarity = Object.values(deepDiveEnemies).find(e => e.name === enemyType)?.rarity || 'Common';
                    switch(rarity) {
                        case 'Common': pearlDrop = Math.floor(Math.random() * 5) + 1; break;
                        case 'Uncommon': pearlDrop = Math.floor(Math.random() * 6) + 10; break;
                        case 'Rare': pearlDrop = Math.floor(Math.random() * 11) + 25; break;
                        case 'Epic': pearlDrop = 40; break;
                        case 'Legendary': 
                            pearlDrop = 50;
                            if (Math.random() < 0.10) pearlDrop += 30;
                            break;
                    }
                    pearlCount += pearlDrop;
                    addNarrativeLine(`The creature dropped ${pearlDrop} pearls.`);
                }
            }

            let currentInventory = isPlayerDeepDiving ? deepDiveInventory : diveInventory;
            const leftover = addToDiveInventory(droppedItem, dropQuantity, currentInventory);
            const collectedAmount = dropQuantity - leftover;

            const modalFn = () => {
                lootModalContent.innerHTML = `
                    <h3>Victory!</h3>
                    <p>The ${enemyType} dropped ${dropQuantity} ${droppedItem}!</p>
                    <p>You collected ${collectedAmount}.</p>
                    ${leftover > 0 ? `<p>(${leftover} was lost, inventory full)</p>` : ''}
                    <button id="close-loot-modal" class="modal-button">Continue Dive</button>
                `;
                lootModal.style.display = 'flex';

                document.getElementById('close-loot-modal').addEventListener('click', () => {
                    lootModal.style.display = 'none';
                    if (isPlayerDeepDiving) renderDeepDiveMinigame(); else renderMinigame();
                }, { once: true });
            };
            showModal(modalFn);
        }
        
        function updateHealthBars() {
            playerHealth = Math.max(0, playerHealth);
            const playerHealthPercent = (playerHealth / MAX_PLAYER_HEALTH) * 100;
            
            // Update all health displays
            playerHealthBar.style.width = `${playerHealthPercent}%`;
            divePlayerHealthBar.style.width = `${playerHealthPercent}%`;
            deepDivePlayerHealthBar.style.width = `${playerHealthPercent}%`;
            playerHealthText.textContent = `${playerHealth} / ${MAX_PLAYER_HEALTH}`;
            diveHealthText.textContent = `${playerHealth} / ${MAX_PLAYER_HEALTH}`;
            deepDiveHealthText.textContent = `${playerHealth} / ${MAX_PLAYER_HEALTH}`;
            
            if (isPlayerInCombat) {
                enemyHealth = Math.max(0, enemyHealth);
                enemyHealthBar.style.width = `${(enemyHealth / maxEnemyHealth) * 100}%`;
                enemyHealthText.textContent = `${enemyHealth} / ${maxEnemyHealth}`;
            }
        }
        
        function addCombatLog(text) {
            combatLog.innerHTML += `<div>${text}</div>`;
            combatLog.scrollTop = combatLog.scrollHeight;
        }

        function updateDiveInventoryDisplay() {
            let html = '';
            for (let i = 0; i < DIVE_INV_SLOTS; i++) {
                const slot = diveInventory[i];
                if (slot) {
                    html += `<div>${slot.item}: ${slot.quantity}/${DIVE_STACK_LIMIT}</div>`;
                } else {
                    html += `<div><i>- Empty -</i></div>`;
                }
            }
            diveInventoryDisplay.innerHTML = html || '<b>Dive Inventory:</b>';
        }

        // --- Admin, Save/Load, Stats, AFK ---
        function checkAdminCode() {
            const code = adminCodeInput.value.toLowerCase();
            const validCodes = {
                'ihategrinding': 'free-crafting-toggle',
                'iamimpatientloyfu': 'fast-cooldowns-toggle',
                'idontneedoxygenbro': 'infinite-moves-toggle',
                'whosgotthesauce': 'god-health-toggle',
                'violenceisnevertheanswer': 'no-enemies-toggle'
            };

            if (validCodes[code]) {
                const commandId = validCodes[code];
                const toggleContainer = document.getElementById(`${commandId}-container`);
                if (toggleContainer) {
                    toggleContainer.style.display = 'flex';
                    unlockedAdminCommands.add(commandId);
                    adminStatus.textContent = `Command unlocked: ${toggleContainer.querySelector('span').textContent}`;
                }
            } else {
                adminStatus.textContent = 'Incorrect Code.';
            }
            adminCodeInput.value = '';
        }
        
        function prepareSaveData() {
            const gameState = {
                resources: { wood: woodCount, stone: stoneCount, fabric: fabricCount, rope: ropeCount, scrap: metallicScrapCount, bandage: bandageCount, pearl: pearlCount, ammo: harpoonAmmoCount, screws: screwsCount },
                purity: { tungsten: tungstenCount, kevlar: kevlarCount, rStone: reinforcedStoneCount, rWood: reinforcedWoodCount },
                deepResources: { coral: coralFragmentsCount, sludge: hydrothermalSludgeCount, sac: bioluminescenceSacCount, filament: electroluminescentFilamentCount, plating: chitinousPlatingCount, shell: nautilusShellCount, viscera: etherealVisceraCount, mineral: hydrothermalVentMineralCount, core: blackIceCoreCount, beak: giantSquidBeakCount, pearl: abyssalPearlCount, tooth: mythicToothCount, bone: boneFragmentCount },
                flags: { search: searchCount, isFree, axeheadFound, hasWoodpile: hasUnlitWoodpile, hasLiquid: hasFlammableLiquid, hasMatch, isSection2, isSection3, fireHint: hasShownFireHint, foundRepairShip: hasFoundRepairShip, foundBattleship: hasFoundBattleship },
                crafted: { gear: hasDivingGear, spear: hasSpear, tank: hasBetterAirTank, gun: hasHarpoonGun, net: hasFishingNet, diveBag: hasDiveBag, furnace: hasFurnace, complexGear: hasComplexDivingGear, kevlarArmour: hasKevlarArmour },
                workers: { total: totalWorkers, idle: idleWorkers, wood: woodGatherers, swim: swimmers, shelters: sheltersBuilt, rShelters: reinforcedSheltersBuilt },
                achievements: Array.from(unlockedAchievements),
                admin: { mode: isAdminMode, free: isFreeCrafting, fast: isCooldownHacked, infMoves: hasInfiniteMoves, godHealth: hasGodHealth, noEnemies: noEnemies, unlocked: Array.from(unlockedAdminCommands) },
                paths: safePaths,
                habitats: { built: habitatsBuilt, placed: placedHabitats },
                stats: gameStats,
                time: gameTime,
                enemies: Array.from(encounteredEnemies),
                nursery: { unlocked: hasUnlockedNursery, eggs: eggInventory, incubators: incubators, pets: Array.from(ownedPets), equipped: equippedPets }
            };
            exportTextarea.value = btoa(JSON.stringify(gameState));
        }
        
        function confirmImport() {
            if (importTextarea.value.trim() !== '') {
                showModal(() => importConfirmModal.style.display = 'flex');
            }
        }
        
        function loadGame() {
            importConfirmModal.style.display = 'none';
            try {
                const savedState = JSON.parse(atob(importTextarea.value));
                
                woodCount = savedState.resources.wood || 0; stoneCount = savedState.resources.stone || 0; fabricCount = savedState.resources.fabric || 0; ropeCount = savedState.resources.rope || 0; metallicScrapCount = savedState.resources.scrap || 0; bandageCount = savedState.resources.bandage || 0; pearlCount = savedState.resources.pearl || 0; harpoonAmmoCount = savedState.resources.ammo || 0; screwsCount = savedState.resources.screws || 0;
                tungstenCount = savedState.purity.tungsten || 0; kevlarCount = savedState.purity.kevlar || 0; reinforcedStoneCount = savedState.purity.rStone || 0; reinforcedWoodCount = savedState.purity.rWood || 0;
                if(savedState.deepResources) {
                    coralFragmentsCount = savedState.deepResources.coral || 0; hydrothermalSludgeCount = savedState.deepResources.sludge || 0; bioluminescenceSacCount = savedState.deepResources.sac || 0; electroluminescentFilamentCount = savedState.deepResources.filament || 0; chitinousPlatingCount = savedState.deepResources.plating || 0; nautilusShellCount = savedState.deepResources.shell || 0; etherealVisceraCount = savedState.deepResources.viscera || 0; hydrothermalVentMineralCount = savedState.deepResources.mineral || 0; blackIceCoreCount = savedState.deepResources.core || 0; giantSquidBeakCount = savedState.deepResources.beak || 0; abyssalPearlCount = savedState.deepResources.pearl || 0; mythicToothCount = savedState.deepResources.tooth || 0; boneFragmentCount = savedState.deepResources.bone || 0;
                }
                searchCount = savedState.flags.search || 0; isFree = savedState.flags.isFree || false; axeheadFound = savedState.flags.axeheadFound || false; hasUnlitWoodpile = savedState.flags.hasWoodpile || false; hasFlammableLiquid = savedState.flags.hasLiquid || false; hasMatch = savedState.flags.hasMatch || false; isSection2 = savedState.flags.isSection2 || false; isSection3 = savedState.flags.isSection3 || false; hasShownFireHint = savedState.flags.fireHint || false; hasFoundRepairShip = savedState.flags.foundRepairShip || false; hasFoundBattleship = savedState.flags.foundBattleship || false;
                hasDivingGear = savedState.crafted.gear || false; hasSpear = savedState.crafted.spear || false; hasBetterAirTank = savedState.crafted.tank || false; hasHarpoonGun = savedState.crafted.gun || false; hasFishingNet = savedState.crafted.net || false; hasDiveBag = savedState.crafted.diveBag || false; hasFurnace = savedState.crafted.furnace || false; hasComplexDivingGear = savedState.crafted.complexGear || false; hasKevlarArmour = savedState.crafted.kevlarArmour || false;
                totalWorkers = savedState.workers.total || 0; woodGatherers = savedState.workers.wood || 0; swimmers = savedState.workers.swim || 0; sheltersBuilt = savedState.workers.shelters || 0; reinforcedSheltersBuilt = savedState.workers.rShelters || 0;
                unlockedAchievements = new Set(savedState.achievements || []);
                if (savedState.admin) {
                    isAdminMode = savedState.admin.mode || false; isFreeCrafting = savedState.admin.free || false; isCooldownHacked = savedState.admin.fast || false; hasInfiniteMoves = savedState.admin.infMoves || false; hasGodHealth = savedState.admin.godHealth || false; noEnemies = savedState.admin.noEnemies || false;
                    unlockedAdminCommands = new Set(savedState.admin.unlocked || []);
                }
                safePaths = savedState.paths || [];
                habitatsBuilt = savedState.habitats.built || 0;
                placedHabitats = savedState.habitats.placed || [];
                gameStats = savedState.stats || { totalWoodCollected: 0, enemiesSlain: {} };
                gameTime = savedState.time || 0;
                encounteredEnemies = new Set(savedState.enemies || []);
                if (savedState.nursery) {
                    hasUnlockedNursery = savedState.nursery.unlocked || false;
                    eggInventory = savedState.nursery.eggs || [];
                    incubators = savedState.nursery.incubators || Array(MAX_INCUBATORS).fill(null);
                    ownedPets = new Set(savedState.nursery.pets || []);
                    equippedPets = savedState.nursery.equipped || [];
                }


                updateUIFromLoad();
                addNarrativeLine("Game loaded successfully!");
                unlockAchievement('welcome_back');
                switchScreen('game');
            } catch (error) {
                addNarrativeLine("Failed to load game. Data may be corrupted.");
                console.error("Load error:", error);
            }
        }
        
        function updateUIFromLoad() {
            updateAllResourceCounts();
            axeItem.style.display = axeheadFound ? 'block' : 'none';
            unlitWoodpileItem.style.display = hasUnlitWoodpile ? 'flex' : 'none';
            flammableLiquidItem.style.display = hasFlammableLiquid ? 'flex' : 'none';
            matchItem.style.display = hasMatch ? 'flex' : 'none';
            divingGearItem.style.display = hasDivingGear ? 'flex' : 'none';
            spearItem.style.display = hasSpear ? 'flex' : 'none';
            airTankItem.style.display = hasBetterAirTank ? 'flex' : 'none';
            harpoonGunItem.style.display = hasHarpoonGun ? 'flex' : 'none';
            fishingNetItem.style.display = hasFishingNet ? 'flex' : 'none';
            diveBagItem.style.display = hasDiveBag ? 'flex' : 'none';
            furnaceItem.style.display = hasFurnace ? 'flex' : 'none';
            complexDivingGearItem.style.display = hasComplexDivingGear ? 'flex' : 'none';
            kevlarArmourItem.style.display = hasKevlarArmour ? 'flex' : 'none';
            if(hasKevlarArmour) DEFAULT_MAX_PLAYER_HEALTH = 25;


            if (axeheadFound) navigationTabs.style.display = 'flex';
            swimButton.style.display = hasMatch ? 'block' : 'none';
            workersTab.style.display = totalWorkers > 0 ? 'block' : 'none';
            diveTab.style.display = hasDivingGear ? 'block' : 'none';
            deepDiveTab.style.display = hasComplexDivingGear ? 'block' : 'none';
            nurseryTab.style.display = hasUnlockedNursery ? 'block' : 'none';
            merchantTab.style.display = hasUnlockedNursery ? 'block' : 'none';
            updateWorkerStats();
            
            // Admin toggles from save
            freeCraftingToggle.checked = isFreeCrafting;
            fastCooldownsToggle.checked = isCooldownHacked;
            infiniteMovesToggle.checked = hasInfiniteMoves;
            godHealthToggle.checked = hasGodHealth;
            noEnemiesToggle.checked = noEnemies;
            unlockedAdminCommands.forEach(id => {
                const container = document.getElementById(`${id}-container`);
                if (container) container.style.display = 'flex';
            });
            
            // Re-add crafting recipes
            document.getElementById('crafting-recipes-container').innerHTML = `<div class="crafting-recipe" id="woodpile-recipe"><p>Unlit Woodpile</p><p>Requires: 5 Wood</p><button id="craft-woodpile-button" class="game-button">Craft</button></div>`;
            document.getElementById('craft-woodpile-button').addEventListener('click', craftWoodpile);
            if (hasUnlitWoodpile) document.getElementById('woodpile-recipe').style.display = 'none';
            if (isSection2) {
                updateShelterRecipe();
                if (!hasDivingGear) addCraftingRecipe(`Simple Diving Gear`, `5 Fabric, 2 Stone, 1 Metallic, 1 Rope`, `craft-diving-gear`, `diving-gear-recipe`, craftDivingGear);
                if (!hasDiveBag) addCraftingRecipe('Dive Bag', '10 Fabric, 5 Rope', 'craft-dive-bag', 'dive-bag-recipe', craftDiveBag, 'Increases dive inventory by 4 slots.');
            }
            if (hasDivingGear) {
                addCraftingRecipe('Spear', '2 Wood, 1 Stone, 1 Metallic, 1 Rope', 'craft-spear', 'spear-recipe', craftSpear);
                addCraftingRecipe('Better Air Tank', '5 Stone, 1 Rope, 2 Fabric', 'craft-air-tank', 'air-tank-recipe', craftBetterAirTank);
                addCraftingRecipe('Bandage', '2 Fabric, 1 Wood', 'craft-bandage', 'bandage-recipe', craftBandage);
            }
             if (hasFoundRepairShip) {
                showRepairShipRecipes();
                if (hasFurnace) {
                     document.getElementById('furnace-recipe').style.display = 'none';
                     addCraftingRecipe('Screws', '1 Metallic Scrap, 1 Stone', 'craft-screws', 'screws-recipe', craftScrews, 'Essential for advanced construction.');
                }
                if (habitatsBuilt >= MAX_HABITATS) {
                    document.getElementById('habitat-recipe').style.display = 'none';
                }
            }
            if (hasFoundBattleship) {
                if (!hasHarpoonGun) addCraftingRecipe('Harpoon Gun', '5 Tungsten, 1 Kevlar, 2 R. Stone, 10 Rope', 'craft-harpoon-gun', 'harpoon-gun-recipe', craftHarpoonGun);
                addCraftingRecipe('Harpoon Ammo (x2)', '1 R. Stone, 1 Metallic Scrap', 'craft-harpoon-ammo', 'harpoon-ammo-recipe', craftHarpoonAmmo);
            }
            if (unlockedAchievements.has('deep_diver')) { // Check if dive gear was ever crafted
                 addCraftingRecipe('Fishing Net', '20 Rope', 'craft-fishing-net', 'fishing-net-recipe', craftFishingNet);
                 purityCraftingContainer.style.display = 'block';
                 addCraftingRecipe('Tungsten', '50 Metallic Scrap', 'craft-tungsten', 'tungsten-recipe', craftTungsten, 'purity-recipes-container');
                 addCraftingRecipe('Kevlar', '100 Fabric', 'craft-kevlar', 'kevlar-recipe', craftKevlar, 'purity-recipes-container');
                 addCraftingRecipe('Reinforced Stone', '50 Stone', 'craft-r-stone', 'r-stone-recipe', craftReinforcedStone, 'purity-recipes-container');
                 addCraftingRecipe('Reinforced Wood', '50 Wood', 'craft-r-wood', 'r-wood-recipe', craftReinforcedWood, 'purity-recipes-container');
                 if (kevlarCount > 0 && !hasComplexDivingGear) {
                    addCraftingRecipe('Complex Diving Gear', '2 Kevlar, 5 Tungsten, 10 R. Stone', 'craft-complex-gear', 'complex-diving-gear-recipe', craftComplexDivingGear, 'purity-recipes-container', 'Allows for Deep Dives.');
                 }
                 if (kevlarCount > 0 && !hasKevlarArmour) {
                    addCraftingRecipe('Kevlar Armour', '5 Kevlar, 10 Rope', 'craft-kevlar-armour', 'kevlar-armour-recipe', craftKevlarArmour, 'purity-recipes-container', 'Increases max health to 25.');
                 }
            }
        }

        function updateAllResourceCounts() {
            woodCountSpan.textContent = woodCount; stoneCountSpan.textContent = stoneCount; fabricCountSpan.textContent = fabricCount; ropeCountSpan.textContent = ropeCount; metallicScrapCountSpan.textContent = metallicScrapCount; bandageCountSpan.textContent = bandageCount; pearlCountSpan.textContent = pearlCount; harpoonAmmoCountSpan.textContent = harpoonAmmoCount; tungstenCountSpan.textContent = tungstenCount; kevlarCountSpan.textContent = kevlarCount; reinforcedStoneCountSpan.textContent = reinforcedStoneCount; reinforcedWoodCountSpan.textContent = reinforcedWoodCount; screwsCountSpan.textContent = screwsCount;
            woodItem.style.display = woodCount > 0 ? 'flex' : 'none'; stoneItem.style.display = stoneCount > 0 ? 'flex' : 'none'; fabricItem.style.display = fabricCount > 0 ? 'flex' : 'none'; ropeItem.style.display = ropeCount > 0 ? 'flex' : 'none'; metallicScrapItem.style.display = metallicScrapCount > 0 ? 'flex' : 'none'; bandageItem.style.display = bandageCount > 0 ? 'flex' : 'none'; pearlItem.style.display = pearlCount > 0 ? 'flex' : 'none'; harpoonAmmoItem.style.display = harpoonAmmoCount > 0 ? 'flex' : 'none'; tungstenItem.style.display = tungstenCount > 0 ? 'flex' : 'none'; kevlarItem.style.display = kevlarCount > 0 ? 'flex' : 'none'; reinforcedStoneItem.style.display = reinforcedStoneCount > 0 ? 'flex' : 'none'; reinforcedWoodItem.style.display = reinforcedWoodCount > 0 ? 'flex' : 'none'; screwsItem.style.display = screwsCount > 0 ? 'flex' : 'none';
            // Deep Dive Items
            document.getElementById('coral-fragments-count').textContent = coralFragmentsCount; document.getElementById('coral-fragments-item').style.display = coralFragmentsCount > 0 ? 'flex' : 'none';
            document.getElementById('hydrothermal-sludge-count').textContent = hydrothermalSludgeCount; document.getElementById('hydrothermal-sludge-item').style.display = hydrothermalSludgeCount > 0 ? 'flex' : 'none';
            document.getElementById('bioluminescence-sac-count').textContent = bioluminescenceSacCount; document.getElementById('bioluminescence-sac-item').style.display = bioluminescenceSacCount > 0 ? 'flex' : 'none';
            document.getElementById('electroluminescent-filament-count').textContent = electroluminescentFilamentCount; document.getElementById('electroluminescent-filament-item').style.display = electroluminescentFilamentCount > 0 ? 'flex' : 'none';
            document.getElementById('chitinous-plating-count').textContent = chitinousPlatingCount; document.getElementById('chitinous-plating-item').style.display = chitinousPlatingCount > 0 ? 'flex' : 'none';
            document.getElementById('nautilus-shell-count').textContent = nautilusShellCount; document.getElementById('nautilus-shell-item').style.display = nautilusShellCount > 0 ? 'flex' : 'none';
            document.getElementById('ethereal-viscera-count').textContent = etherealVisceraCount; document.getElementById('ethereal-viscera-item').style.display = etherealVisceraCount > 0 ? 'flex' : 'none';
            document.getElementById('hydrothermal-vent-mineral-count').textContent = hydrothermalVentMineralCount; document.getElementById('hydrothermal-vent-mineral-item').style.display = hydrothermalVentMineralCount > 0 ? 'flex' : 'none';
            document.getElementById('black-ice-core-count').textContent = blackIceCoreCount; document.getElementById('black-ice-core-item').style.display = blackIceCoreCount > 0 ? 'flex' : 'none';
            document.getElementById('giant-squid-beak-count').textContent = giantSquidBeakCount; document.getElementById('giant-squid-beak-item').style.display = giantSquidBeakCount > 0 ? 'flex' : 'none';
            document.getElementById('abyssal-pearl-count').textContent = abyssalPearlCount; document.getElementById('abyssal-pearl-item').style.display = abyssalPearlCount > 0 ? 'flex' : 'none';
            document.getElementById('mythic-tooth-count').textContent = mythicToothCount; document.getElementById('mythic-tooth-item').style.display = mythicToothCount > 0 ? 'flex' : 'none';
            document.getElementById('bone-fragment-count').textContent = boneFragmentCount; document.getElementById('bone-fragment-item').style.display = boneFragmentCount > 0 ? 'flex' : 'none';
        }

        function updateStatisticsScreen() {
            statsContainer.innerHTML = `
                <h2>Resources</h2>
                <p>Total Wood Gathered: ${gameStats.totalWoodCollected || 0}</p>
                <p>Total Stone Gathered: ${gameStats.totalStoneCollected || 0}</p>
                <p>Total Fabric Gathered: ${gameStats.totalFabricCollected || 0}</p>
                <p>Total Rope Gathered: ${gameStats.totalRopeCollected || 0}</p>
                <p>Total Metallic Scrap Gathered: ${gameStats.totalMetallicScrapCollected || 0}</p>
                <h2>Combat</h2>
                <p>Total Dives: ${gameStats.totalDives || 0}</p>
                <p>Total Deep Dives: ${gameStats.totalDeepDives || 0}</p>
            `;
            let enemyStatsHTML = '<h3>Enemies Slain:</h3>';
            for (const enemy in gameStats.enemiesSlain) {
                enemyStatsHTML += `<p>${enemy}: ${gameStats.enemiesSlain[enemy]}</p>`;
            }
            statsContainer.innerHTML += enemyStatsHTML;
        }
        
        function updateBestiaryScreen() {
            bestiaryContainer.innerHTML = '';
            if (encounteredEnemies.size === 0) {
                bestiaryContainer.innerHTML = '<p>You have not encountered any enemies yet.</p>';
                return;
            }
            
            const allEnemies = {...enemies, ...deepDiveEnemies};
            for (const enemyId of encounteredEnemies) {
                const enemy = allEnemies[enemyId];
                if (enemy) {
                    const entry = document.createElement('div');
                    entry.classList.add('bestiary-entry');
                    entry.innerHTML = `
                        <h3>${enemy.name}</h3>
                        <p><b>Health:</b> ${enemy.health}</p>
                        <p><b>Damage:</b> ${enemy.minDmg} - ${enemy.maxDmg}</p>
                        <p><b>Attack Speed:</b> ${enemy.cooldown / 1000} seconds</p>
                        ${enemy.special ? `<p><b>Special:</b> ${enemy.special.description}</p>` : ''}
                        ${enemy.special2 ? `<p><b>Special 2:</b> ${enemy.special2.description}</p>` : ''}
                    `;
                    bestiaryContainer.appendChild(entry);
                }
            }
        }

        function resetAfkTimer() {
            clearTimeout(afkTimer);
            afkTimer = setTimeout(() => {
                isGamePaused = true;
                showModal(() => afkModal.style.display = 'flex');
                unlockAchievement('afk_lol');
            }, AFK_TIMEOUT);
        }
        
        // --- Timer Logic ---
        function startTimer() {
            if (timerInterval) clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                if (!isGamePaused) {
                    gameTime++;
                    updateTimerDisplay();
                }
            }, 1000);
        }

        function updateTimerDisplay() {
            const hours = Math.floor(gameTime / 3600).toString().padStart(2, '0');
            const minutes = Math.floor((gameTime % 3600) / 60).toString().padStart(2, '0');
            const seconds = (gameTime % 60).toString().padStart(2, '0');
            speedrunTimer.textContent = `${hours}:${minutes}:${seconds}`;
        }
        
         // --- Habitat Logic ---
        function buildHabitat() {
            if (habitatsBuilt > placedHabitats.length) {
                placedHabitats.push({ x: playerPosition.x, y: playerPosition.y, usedThisDive: false });
                addNarrativeLine("You've placed an Underwater Habitat.");
                if (!isSection3) {
                    isSection3 = true;
                    unlockAchievement('deep_sea_dweller');
                }
                renderMinigame();
            }
        }
        
        function useHabitat(habitat) {
            habitat.usedThisDive = true;
            let airRestored = 20;
            if(hasBetterAirTank) airRestored = 40; // Base 20 + Tank 20
            diveMoves += airRestored;
            
            const healedAmount = Math.min(10, MAX_PLAYER_HEALTH - playerHealth);
            playerHealth += healedAmount;

            addNarrativeLine(`Habitat restored ${airRestored} moves and healed ${healedAmount} HP.`);
            renderMinigame();
        }
        
        function showRepairShipRecipes() {
            if (!document.getElementById('furnace-recipe')) {
                addCraftingRecipe('Furnace', '10 Metallic Scrap, 2 Tungsten, 50 Wood', 'craft-furnace', 'furnace-recipe', craftFurnace, 'crafting-recipes-container', 'Allows for smelting and creating advanced components.');
            }
            if (!document.getElementById('habitat-recipe')) {
                 addCraftingRecipe('Underwater Habitat', '3 Tungsten, 45 Metallic Scrap, 100 Screws', 'craft-habitat', 'habitat-recipe', craftUnderwaterHabitat, 'crafting-recipes-container', 'A small, safe outpost for deep-sea exploration.');
            }
        }
        
        function playClickSound() {
            clickSound.currentTime = 0;
            clickSound.play();
        }
        
        // --- Deep Dive Logic ---
        
        const deepDiveEnemies = {
            snipeEel: { name: 'Snipe Eel', health: 10, minDmg: 1, maxDmg: 2, cooldown: 2000, attackName: 'Bite', rarity: 'Common', drops: [{item: 'Coral Fragments', min: 1, max: 3}, {item: 'Hydrothermal Sludge', min: 1, max: 2}] },
            squidworm: { name: 'Squidworm', health: 20, minDmg: 0, maxDmg: 0, cooldown: 2000, attackName: 'Wiggle', rarity: 'Common', special: { name: 'Tentacle Barrage', cooldown: 2000, damage: 1, hits: 4, description: 'Launches a flurry of worm-like tentacles.' }, drops: [{item: 'Coral Fragments', min: 2, max: 4}, {item: 'Bio-Luminescence Sac', min: 1, max: 1}] },
            dragonfish: { name: 'Dragonfish', health: 25, minDmg: 3, maxDmg: 3, cooldown: 3000, attackName: 'Bite', rarity: 'Uncommon', special: { name: 'Bioluminescent Flash', cooldown: 8000, damage: 2, stun: true, description: 'Flashes its light organ, stunning the player.' }, drops: [{item: 'Bio-Luminescence Sac', min: 1, max: 2}, {item: 'Electroluminescent Filament', min: 1, max: 1}] },
            octopusSquid: { name: 'Octopus Squid', health: 30, minDmg: 0, maxDmg: 0, cooldown: 3000, attackName: 'Stare', rarity: 'Uncommon', special: { name: 'Jet Propulsion Burst', cooldown: 3000, damage: 7, description: 'Uses a powerful jet to ram the player.' }, drops: [{item: 'Chitinous Plating', min: 1, max: 1}, {item: 'Electroluminescent Filament', min: 1, max: 2}] },
            bigfinSquid: { name: 'Bigfin Squid', health: 35, minDmg: 2, maxDmg: 3, cooldown: 3000, attackName: 'Whip', rarity: 'Uncommon', special: { name: 'Trawl Net', cooldown: 7000, damage: 5, description: 'Extends its long arms in a cone-shaped attack.' }, drops: [{item: 'Chitinous Plating', min: 1, max: 2}, {item: 'Bio-Luminescence Sac', min: 1, max: 2}] },
            frilledShark: { name: 'Frilled Shark', health: 40, minDmg: 3, maxDmg: 4, cooldown: 2500, attackName: 'Bite', rarity: 'Uncommon', special: { name: 'Serpent Lunge', cooldown: 5000, damage: 7, description: 'Coils and then lunges with its jaw wide open.' }, drops: [{item: 'Chitinous Plating', min: 2, max: 3}, {item: 'Hydrothermal Sludge', min: 1, max: 3}] },
            ghostshark: { name: 'Ghostshark', health: 45, minDmg: 4, maxDmg: 5, cooldown: 3000, attackName: 'Phase Bite', rarity: 'Rare', special: { name: 'Spectral Phase', cooldown: 9000, damage: 8, description: 'Turns semi-transparent and charges through the player.' }, drops: [{item: 'Nautilus Shell', min: 1, max: 1}, {item: 'Ethereal Viscera', min: 1, max: 2}] },
            giantPhantomJelly: { name: 'Giant Phantom Jelly', health: 50, minDmg: 1, maxDmg: 2, cooldown: 2000, attackName: 'Brush', rarity: 'Rare', special: { name: 'Ethereal Envelopment', cooldown: 4000, damage: 4, duration: 3, description: 'Envelops the player, dealing damage over time.' }, drops: [{item: 'Ethereal Viscera', min: 1, max: 3}, {item: 'Bio-Luminescence Sac', min: 2, max: 3}] },
            deepseaLizardfish: { name: 'Deep-sea Lizardfish', health: 50, minDmg: 5, maxDmg: 6, cooldown: 4000, attackName: 'Chomp', rarity: 'Rare', special: { name: 'Impale', cooldown: 10000, damage: 9, description: 'Dashes forward, impaling the player.' }, drops: [{item: 'Hydrothermal Sludge', min: 2, max: 4}, {item: 'Nautilus Shell', min: 1, max: 1}] },
            goblinShark: { name: 'Goblin Shark', health: 55, minDmg: 6, maxDmg: 7, cooldown: 3500, attackName: 'Bite', rarity: 'Epic', special: { name: 'Jaw Protrusion', cooldown: 12000, damage: 9, description: 'Rapidly extends its jaw forward in a surprise attack.' }, drops: [{item: 'Hydrothermal Vent Mineral', min: 1, max: 1}, {item: 'Black Ice Core', min: 1, max: 1}] },
            oarfish: { name: 'Oarfish', health: 60, minDmg: 5, maxDmg: 5, cooldown: 2000, attackName: 'Slap', rarity: 'Epic', special: { name: 'Serpent\'s Whip', cooldown: 4000, damage: 10, description: 'Lashes out with its immense tail.' }, drops: [{item: 'Black Ice Core', min: 1, max: 2}, {item: 'Chitinous Plating', min: 2, max: 4}] },
            giantIsopod: { name: 'Giant Isopod', health: 35, minDmg: 2, maxDmg: 3, cooldown: 2500, attackName: 'Nip', rarity: 'Epic', special: { name: 'Defensive Roll', cooldown: 10000, damage: 6, description: 'Curls into a spiky ball and rolls at the player.' }, drops: [{item: 'Hydrothermal Vent Mineral', min: 1, max: 2}, {item: 'Chitinous Plating', min: 3, max: 5}] },
            greatWhite: { name: 'Great White', health: 80, minDmg: 6, maxDmg: 7, cooldown: 3000, attackName: 'Bite', rarity: 'Legendary', special: { name: 'Apex Charge', cooldown: 8000, damage: 8, description: 'Charges directly at the player.' }, special2: { name: 'Tearing Bite', cooldown: 15000, damage: 10, description: 'A massive, tearing bite.' }, drops: [{item: 'Giant Squid Beak', min: 1, max: 1}, {item: 'Abyssal Pearl', min: 1, max: 1}, {item: 'Mythic Tooth', min: 1, max: 1}] },
            whaleShark: { name: 'Whale Shark', health: 75, minDmg: 1, maxDmg: 2, cooldown: 1500, attackName: 'Nudge', rarity: 'Legendary', special: { name: 'Vortex Gulp', cooldown: 12000, damage: 2, duration: 5, description: 'Creates a vortex, pulling the player in for damage.' }, drops: [{item: 'Abyssal Pearl', min: 1, max: 2}, {item: 'Bone Fragment', min: 1, max: 3}, {item: 'Mythic Tooth', min: 1, max: 1}] },
            colossalSquid: { name: 'Giant Colossal Squid', health: 80, minDmg: 7, maxDmg: 8, cooldown: 2500, attackName: 'Arm Slap', rarity: 'Legendary', special: { name: 'Beak Rend', cooldown: 6000, damage: 10, description: 'Lunges forward with its razor-sharp beak.' }, special2: { name: 'Ink Cloud', cooldown: 15000, damage: 2, description: 'Releases a blinding cloud of ink.' }, drops: [{item: 'Giant Squid Beak', min: 1, max: 2}, {item: 'Bone Fragment', min: 2, max: 4}, {item: 'Abyssal Pearl', min: 1, max: 1}] },
        };
        
        const deepDiveLandmarks = [
            { name: 'Hydrothermal vent valley', lootTable: ['Hydrothermal Sludge', 'Hydrothermal Vent Mineral'] },
            { name: 'Semi-Submersible Rig Base', lootTable: ['Metallic Scrap', 'Chitinous Plating', 'Electroluminescent Filament'] },
            { name: 'Abandoned Pipelines', lootTable: ['Hydrothermal Sludge', 'Metallic Scrap', 'Nautilus Shell'] },
            { name: 'Abyssal Plain', lootTable: ['Bone Fragment', 'Ethereal Viscera', 'Coral Fragments'] },
            { name: 'Whale falls', lootTable: ['Bone Fragment', 'Bio-Luminescence Sac', 'Chitinous Plating'] }
        ];

        function getEnemyByChance() {
            const rand = Math.random() * 100;
            if (rand < 1) return deepDiveEnemies.colossalSquid;
            if (rand < 3) return deepDiveEnemies.whaleShark;
            if (rand < 6) return deepDiveEnemies.greatWhite;
            if (rand < 14) return deepDiveEnemies.oarfish;
            if (rand < 24) return deepDiveEnemies.giantIsopod;
            if (rand < 34) return deepDiveEnemies.goblinShark;
            if (rand < 54) return deepDiveEnemies.deepseaLizardfish;
            if (rand < 79) return deepDiveEnemies.giantPhantomJelly;
            if (rand < 109) return deepDiveEnemies.ghostshark; // This logic is cumulative, so I'll adjust
            // Let's use a weighted list approach instead
            const encounterTable = [
                { enemy: 'snipeEel', weight: 80 },
                { enemy: 'squidworm', weight: 65 },
                { enemy: 'dragonfish', weight: 50 },
                { enemy: 'octopusSquid', weight: 40 },
                { enemy: 'bigfinSquid', weight: 40 },
                { enemy: 'frilledShark', weight: 45 },
                { enemy: 'ghostshark', weight: 30 },
                { enemy: 'giantPhantomJelly', weight: 25 },
                { enemy: 'deepseaLizardfish', weight: 20 },
                { enemy: 'goblinShark', weight: 10 },
                { enemy: 'oarfish', weight: 8 },
                { enemy: 'giantIsopod', weight: 10 },
                { enemy: 'greatWhite', weight: 3 },
                { enemy: 'whaleShark', weight: 2 },
                { enemy: 'colossalSquid', weight: 1 },
            ];
            const totalWeight = encounterTable.reduce((sum, item) => sum + item.weight, 0);
            let random = Math.random() * totalWeight;
            for (const item of encounterTable) {
                if (random < item.weight) {
                    return deepDiveEnemies[item.enemy];
                }
                random -= item.weight;
            }
        }
        
        function updateDeepDivePrepUI() {
            deepToolSelectionContainer.innerHTML = '';
            if (hasSpear) deepToolSelectionContainer.innerHTML += `<div><input type="checkbox" id="deep-select-spear" value="spear"><label for="deep-select-spear"> Bring Spear</label></div>`;
            if (hasHarpoonGun) deepToolSelectionContainer.innerHTML += `<div><input type="checkbox" id="deep-select-harpoon" value="harpoon"><label for="deep-select-harpoon"> Bring Harpoon Gun (${harpoonAmmoCount} ammo)</label></div>`;
            if (hasBetterAirTank) deepToolSelectionContainer.innerHTML += `<div><input type="checkbox" id="deep-select-air-tank" value="air-tank"><label for="deep-select-air-tank"> Bring Better Air Tank (+15 Moves)</label></div>`;
            if (hasFishingNet) deepToolSelectionContainer.innerHTML += `<div><input type="checkbox" id="deep-select-net" value="net"><label for="deep-select-net"> Bring Fishing Net</label></div>`;
            if (bandageCount > 0) deepToolSelectionContainer.innerHTML += `<div><label for="deep-select-bandages">Bring Bandages: </label><input type="number" id="deep-select-bandages" value="0" min="0" max="${bandageCount}" style="width: 50px;"><span> (Max: ${bandageCount})</span></div>`;
            if (deepToolSelectionContainer.innerHTML === '') deepToolSelectionContainer.innerHTML = '<p>You have no special tools to bring.</p>';
        }
        
        function beginDeepDive() {
            if (beginDeepDiveButton.classList.contains('disabled')) return;

            isPlayerDeepDiving = true;
            gameStats.totalDeepDives++;
            deepDivePrepArea.style.display = 'none';
            deepDiveMinigameArea.style.display = 'flex';
            
            MAX_PLAYER_HEALTH = hasGodHealth ? 500 : (hasKevlarArmour ? 25 : 15);
            playerHealth = MAX_PLAYER_HEALTH;
            deepDiveMoves = 5;
            deepDiveTools = [];

            const spearCheckbox = document.getElementById('deep-select-spear');
            const airTankCheckbox = document.getElementById('deep-select-air-tank');
            const netCheckbox = document.getElementById('deep-select-net');
            const harpoonCheckbox = document.getElementById('deep-select-harpoon');
            const bandageInput = document.getElementById('deep-select-bandages');
            
            deepDiveBandages = 0;
            if (bandageInput && parseInt(bandageInput.value) > 0) {
                const bandagesToTake = Math.min(bandageCount, parseInt(bandageInput.value));
                deepDiveBandages = bandagesToTake;
                bandageCount -= bandagesToTake;
                bandageCountSpan.textContent = bandageCount;
            }

            if (airTankCheckbox && airTankCheckbox.checked) { deepDiveMoves += 15; deepDiveTools.push('air-tank'); }
            if (spearCheckbox && spearCheckbox.checked) { deepDiveTools.push('spear'); }
            if (netCheckbox && netCheckbox.checked) { deepDiveTools.push('net'); }
            if (harpoonCheckbox && harpoonCheckbox.checked) { deepDiveTools.push('harpoon'); }

            deepDiveInventory = [];
            generateDeepDiveMap();
            renderDeepDiveMinigame();
            document.addEventListener('keydown', handlePlayerMovement);
        }
        
        function generateDeepDiveMap() {
            deepMinigameMap = Array(MAP_SIZE).fill(null).map(() => Array(MAP_SIZE).fill({ type: 'path' }));
            const center = Math.floor(MAP_SIZE / 2);
            
            const totalLandmarks = 15; // Fewer landmarks in the deep
            for (let i = 0; i < totalLandmarks; i++) {
                const x = Math.floor(Math.random() * MAP_SIZE);
                const y = Math.floor(Math.random() * MAP_SIZE);
                const landmarkType = deepDiveLandmarks[Math.floor(Math.random() * deepDiveLandmarks.length)];
                if (x !== center || y !== center) deepMinigameMap[y][x] = { type: 'landmark', details: landmarkType };
            }

            deepMinigameMap[center][center] = { type: 'boat' };
            deepPlayerPosition = { x: center, y: center };
        }
        
        function renderDeepDiveMinigame() {
            let gridHTML = '';
            for (let y = 0; y < MAP_SIZE; y++) {
                 let rowHTML = '';
                for (let x = 0; x < MAP_SIZE; x++) {
                    const tile = deepMinigameMap[y][x];
                    if (x === deepPlayerPosition.x && y === deepPlayerPosition.y) {
                        let playerIcon = '';
                        if (equippedPets[0]) playerIcon += `<span class="pet">${equippedPets[0].icon}</span>`;
                        playerIcon += '<span class="player">@</span>';
                        if (equippedPets[1]) playerIcon += `<span class="pet">${equippedPets[1].icon}</span>`;
                        rowHTML += playerIcon;
                    }
                    else if (tile.type === 'boat') rowHTML += '<span class="boat">$</span>';
                    else if (tile.type === 'landmark') rowHTML += `<span class="landmark" data-name="${tile.details.name}">#<span class="tooltip-text">${tile.details.name}</span></span>`;
                    else rowHTML += '*';
                }
                 gridHTML += rowHTML + '\n';
            }
            deepMinigameGrid.innerHTML = gridHTML;
            deepMovesLeftSpan.textContent = hasInfiniteMoves ? '∞' : deepDiveMoves;
            updateHealthBars();
            deepDiveBandageCountSpan.textContent = deepDiveBandages;
            deepUseBandageButton.style.display = deepDiveBandages > 0 ? 'inline-block' : 'none';
            updateDeepDiveInventoryDisplay();
        }
        
        function handleDeepDiveMovement(e) {
            let newX = deepPlayerPosition.x, newY = deepPlayerPosition.y;
            if (e.key === 'ArrowUp' || e.key.toLowerCase() === 'w') newY--;
            else if (e.key === 'ArrowDown' || e.key.toLowerCase() === 's') newY++;
            else if (e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a') newX--;
            else if (e.key === 'ArrowRight' || e.key.toLowerCase() === 'd') newX++;
            else return;

            if (newY >= 0 && newY < MAP_SIZE && newX >= 0 && newX < MAP_SIZE) {
                deepPlayerPosition = { x: newX, y: newY };
                if (!hasInfiniteMoves) deepDiveMoves--;

                const currentTile = deepMinigameMap[newY][newX];

                if (currentTile.type === 'path' && !noEnemies && Math.random() < 0.30) {
                    startCombat(getEnemyByChance());
                    return;
                } else if (currentTile.type === 'landmark') {
                    openLandmark(currentTile.details, true);
                    deepMinigameMap[newY][newX] = { type: 'path' };
                } else if (currentTile.type === 'boat') {
                    endDeepDive(true);
                    return;
                }

                renderDeepDiveMinigame();
                if (deepDiveMoves <= 0 && !hasInfiniteMoves) endDeepDive(false);
            }
        }
        
        function endDeepDive(isSuccess) {
            document.removeEventListener('keydown', handlePlayerMovement);
            isPlayerDeepDiving = false;
            deepDiveMinigameArea.style.display = 'none';
            deepDivePrepArea.style.display = 'flex';
            startCooldown(beginDeepDiveButton, deepDiveCooldownBar, deepDiveCooldownDuration);
            MAX_PLAYER_HEALTH = hasKevlarArmour ? 25 : 15; // Reset health
            
            bandageCount += deepDiveBandages; // Return unused bandages
            bandageCountSpan.textContent = bandageCount;
            
            if (isSuccess) {
                addNarrativeLine("You surfaced from the abyss, alive.");
                deepDiveInventory.forEach(slot => {
                    if (slot) {
                        addItemToInventory(slot.item, slot.quantity);
                        addNarrativeLine(`Brought back ${slot.quantity} ${slot.item}.`);
                    }
                });
            } else {
                addNarrativeLine("The pressure was too much... You blacked out and lost everything from the deep dive.");
            }
            deepDiveInventory = [];
            deepDiveTools = [];
            
            switchScreen('game');
            processQueuedEvents();
            processPopupQueue();
        }
        
        function updateDeepDiveInventoryDisplay() {
            let html = '';
            for (let i = 0; i < DIVE_INV_SLOTS; i++) {
                const slot = deepDiveInventory[i];
                if (slot) {
                    html += `<div>${slot.item}: ${slot.quantity}/${DIVE_STACK_LIMIT}</div>`;
                } else {
                    html += `<div><i>- Empty -</i></div>`;
                }
            }
            deepDiveInventoryDisplay.innerHTML = html || '<b>Deep Dive Inventory:</b>';
        }
        
        function handleEnemySpecial(specialName) {
            const special = currentEnemy.special.name === specialName ? currentEnemy.special : currentEnemy.special2;
            addCombatLog(`The ${currentEnemy.name} uses ${special.name}!`);

            switch(special.name) {
                case 'Tentacle Barrage':
                    for(let i = 0; i < special.hits; i++) {
                        playerHealth -= special.damage;
                        addCombatLog(`A tentacle hits you for ${special.damage} damage.`);
                    }
                    break;
                case 'Bioluminescent Flash':
                    playerHealth -= special.damage;
                    isPlayerStunned = true;
                    addCombatLog(`The flash stuns you!`);
                    setTimeout(() => { if(!isGamePaused) { isPlayerStunned = false; addCombatLog("You can move again."); } }, 2000);
                    break;
                case 'Ethereal Envelopment':
                case 'Vortex Gulp':
                    playerDoT = { damage: special.damage, duration: special.duration };
                    addCombatLog(`You are caught in the attack!`);
                    break;
                default:
                    playerHealth -= special.damage;
                    break;
            }
            updateHealthBars();
            if (playerHealth <= 0) endCombat('loss');
        }

        // --- Nursery and Merchant Logic ---
        function updateMerchantUI() {
            pearlDisplay.textContent = `Pearls: ${pearlCount}`;
            merchantEggsContainer.innerHTML = ''; // Clear existing eggs

            merchantEggs.forEach(egg => {
                const eggDiv = document.createElement('div');
                eggDiv.classList.add('egg-for-sale');
                eggDiv.innerHTML = `
                    <div class="egg-art">${egg.art}</div>
                    <div>${egg.name}</div>
                    <div>Cost: ${egg.cost} Pearls</div>
                `;
                eggDiv.addEventListener('click', () => buyEgg(egg.type, egg.cost));
                merchantEggsContainer.appendChild(eggDiv);
            });
        }

        function buyEgg(type, cost) {
            if (pearlCount >= cost) {
                pearlCount -= cost;
                const newEgg = { type: type, id: Date.now() };
                eggInventory.push(newEgg);
                addNarrativeLine(`You purchased a ${type} Egg!`);
                updateMerchantUI(); // Refresh display to show new pearl count
                updateNurseryUI(); // Refresh nursery to show the new egg
            } else {
                addNarrativeLine("You don't have enough pearls to buy this egg.");
            }
        }


        // --- Main Game Event Listeners ---
        mainButton.addEventListener('click', progressGame);
        knifeItem.addEventListener('click', openKnifeModal);
        cutRopeButton.addEventListener('click', cutTheRope);
        craftWoodpileButton.addEventListener('click', craftWoodpile);
        gatherWoodButton.addEventListener('click', gatherWood);
        takeInPersonButton.addEventListener('click', recruitNewPerson);
        takeInPeopleButton.addEventListener('click', recruitNewPeople);
        assignWoodGathererButton.addEventListener('click', () => assignWorker('wood'));
        assignSwimmerButton.addEventListener('click', () => assignWorker('swim'));
        recallWoodGathererButton.addEventListener('click', () => recallWorker('wood'));
        recallSwimmerButton.addEventListener('click', () => recallWorker('swim'));
        unlitWoodpileItem.addEventListener('click', () => handleInventoryClick(unlitWoodpileItem));
        matchItem.addEventListener('click', () => handleInventoryClick(matchItem));
        lightFireButton.addEventListener('click', lightThePile);
        gameTab.addEventListener('click', () => switchScreen('game'));
        resourceTab.addEventListener('click', () => switchScreen('resource'));
        craftingTab.addEventListener('click', () => switchScreen('crafting'));
        achievementsTab.addEventListener('click', () => switchScreen('achievements'));
        workersTab.addEventListener('click', () => switchScreen('workers'));
        diveTab.addEventListener('click', () => switchScreen('dive'));
        deepDiveTab.addEventListener('click', () => switchScreen('deep-dive'));
        nurseryTab.addEventListener('click', () => switchScreen('nursery'));
        merchantTab.addEventListener('click', () => switchScreen('merchant'));
        progressionButton.addEventListener('click', () => switchScreen('progression'));
        progressionBackButton.addEventListener('click', () => switchScreen('game'));
        commandsButton.addEventListener('click', () => switchScreen('commands'));
        commandsBackButton.addEventListener('click', () => switchScreen('game'));
        adminSubmitButton.addEventListener('click', checkAdminCode);
        freeCraftingToggle.addEventListener('change', () => { isFreeCrafting = freeCraftingToggle.checked; if(isFreeCrafting) unlockAchievement('god_mode'); });
        fastCooldownsToggle.addEventListener('change', () => { isCooldownHacked = fastCooldownsToggle.checked; });
        infiniteMovesToggle.addEventListener('change', () => { hasInfiniteMoves = infiniteMovesToggle.checked; });
        godHealthToggle.addEventListener('change', () => { hasGodHealth = godHealthToggle.checked; });
        noEnemiesToggle.addEventListener('change', () => { noEnemies = noEnemiesToggle.checked; });
        saveButton.addEventListener('click', () => switchScreen('save'));
        saveLoadBackButton.addEventListener('click', () => switchScreen('game'));
        importButton.addEventListener('click', confirmImport);
        confirmImportButton.addEventListener('click', loadGame);
        cancelImportButton.addEventListener('click', () => { importConfirmModal.style.display = 'none'; });
        swimButton.addEventListener('click', swim);
        beginDiveButton.addEventListener('click', beginDive);
        beginDeepDiveButton.addEventListener('click', beginDeepDive);
        useBandageButton.addEventListener('click', useBandage);
        deepUseBandageButton.addEventListener('click', useBandage);
        buildHabitatButton.addEventListener('click', buildHabitat);
        fistAttackButton.addEventListener('click', () => playerAttack('fist'));
        spearAttackButton.addEventListener('click', () => playerAttack('spear'));
        harpoonAttackButton.addEventListener('click', () => playerAttack('harpoon'));
        tangleButton.addEventListener('click', playerTangle);
        healButton.addEventListener('click', useBandage);
        fleeButton.addEventListener('click', attemptFlee);
        musicControlBtn.addEventListener('click', toggleMusic);
        document.getElementById('achievements-button').addEventListener('click', () => switchScreen('achievements'));
        document.getElementById('back-button').addEventListener('click', () => switchScreen('game'));
        statisticsButton.addEventListener('click', () => switchScreen('statistics'));
        statisticsBackButton.addEventListener('click', () => switchScreen('game'));
        bestiaryButton.addEventListener('click', () => switchScreen('bestiary'));
        bestiaryBackButton.addEventListener('click', () => switchScreen('statistics'));
        
        document.querySelectorAll('.battleship-choice-button').forEach(button => {
            button.addEventListener('click', (e) => {
                const item = e.target.dataset.item;
                addItemToInventory(item, 1);
                addNarrativeLine(`You collected 1 ${item} from the battleship.`);
                battleshipChoiceModal.style.display = 'none';
                renderMinigame();
            });
        });
        
        resumeGameButton.addEventListener('click', () => {
            isGamePaused = false;
            afkModal.style.display = 'none';
            resetAfkTimer();
            processPopupQueue();
        });
        
        // Add click sound to all buttons
        document.querySelectorAll('button').forEach(button => {
            button.addEventListener('click', playClickSound);
        });

        window.addEventListener('mousemove', resetAfkTimer);
        window.addEventListener('keydown', resetAfkTimer);
        window.addEventListener('click', resetAfkTimer);

        // Initial setup
        const initialNarrative = document.querySelector('.narrative-line');
        void initialNarrative.offsetWidth;
        initialNarrative.style.opacity = '1';
        initialNarrative.style.transform = 'translateY(0)';
        bouncingText.innerHTML = bouncingText.textContent.split('').map(letter => `<span>${letter === ' ' ? '&nbsp;' : letter}</span>`).join('');
        bouncingText.querySelectorAll('span').forEach((span, index) => { span.style.animationDelay = `${index * 0.05}s`; });
        document.body.addEventListener('click', () => { if (!isMusicPlaying && backgroundMusic.paused) toggleMusic(); }, { once: true });
        resetAfkTimer();
        startTimer();

    </script>
// --- PETS DATA ---
const PETS = {
  // Add all pets and their perks from your game here (see previous answer for complete list)
  "Nudibranch": { rarity: "Common", perk: "Adaptive Defense" },
  // ... all other pets ...
  "Immortal Jellyfish": { rarity: "Secret", perk: "Rejuvenation Cycle" },
};

// --- STATUS TRACKING HELPERS ---
function setStatus(statusObj, effect, durationSeconds) {
  statusObj[effect] = Date.now() + durationSeconds * 1000;
}
function hasStatus(statusObj, effect) {
  return statusObj[effect] && Date.now() < statusObj[effect];
}
function clearExpiredStatus(statusObj) {
  Object.keys(statusObj).forEach(key => {
    if (statusObj[key] < Date.now()) delete statusObj[key];
  });
}

// --- MAIN PET PERK INTEGRATION ---
function applyPetPerks(gameState, playerStatus, enemyStatus, combatContext) {
  // combatContext: { round, isMelee, isCritical, attackDirection, etc. }
  // playerStatus and enemyStatus are objects for status effects (e.g. poison, shield, etc.)

  gameState.equippedPets.forEach(petName => {
    const pet = PETS[petName];
    if (!pet) return;

    switch (pet.perk) {
      // --- COMMON ---
      case "Adaptive Defense":
        if (combatContext.enemySpecial && Math.random() < 0.05)
          combatContext.enemyAttackMiss = true;
        break;
      case "Swift Current":
        gameState.diveMoves += 5;
        break;
      case "Regenerative Aid":
        if (combatContext.round % 5 === 0)
          gameState.playerHealth = Math.min(gameState.playerMaxHealth, gameState.playerHealth + 1);
        break;
      case "Slime Trail":
        if (combatContext.playerWasAttacked) {
          setStatus(enemyStatus, "slowed", 2);
          gameState.enemyAttackCooldown *= 1.1;
        }
        break;
      case "Fortify":
        gameState.playerMaxHealth = Math.floor(gameState.playerMaxHealth * 1.1);
        break;
      case "Symbiotic Guard":
        if (!playerStatus.shieldActive) playerStatus.shieldActive = 3;
        break;
      case "Piercing Swarm":
        combatContext.damage += 1;
        break;
      case "Cooldown Link 10":
        gameState.cooldownMultiplier = (gameState.cooldownMultiplier || 1) * 0.9;
        break;
      case "Stealthy Current":
        gameState.enemyHitChance = (gameState.enemyHitChance || 1) * 0.9;
        break;
      case "Spiky Aura":
        if (combatContext.playerWasAttacked) gameState.enemyHealth -= 1;
        break;

      // --- UNCOMMON ---
      case "Deep Descent":
        if (!hasStatus(playerStatus, "deepDescent"))
          setStatus(playerStatus, "deepDescent", 15);
        break;
      case "Invisibility Ink":
        setStatus(playerStatus, "invisible", 2);
        break;
      case "Cooldown Link 20":
        gameState.cooldownMultiplier = (gameState.cooldownMultiplier || 1) * 0.8;
        break;
      case "Protective Shell":
        if (!playerStatus.shellActive) playerStatus.shellActive = 8;
        break;
      case "Enduring Shell":
        setStatus(playerStatus, "defenseBoost", 8);
        break;
      case "Camouflage":
        gameState.enemyHitChance = (gameState.enemyHitChance || 1) * 0.85;
        break;
      case "Evasive Leap":
        playerStatus.evasiveLeap = 5;
        break;
      case "Quick-Heal":
        gameState.healingItemEffect = (gameState.healingItemEffect || 1) * 1.2;
        break;
      case "Sanguine Strike":
        if (Math.random() < 0.1)
          gameState.playerHealth = Math.min(gameState.playerMaxHealth, gameState.playerHealth + combatContext.damage);
        break;
      case "Venomous Touch":
        if (combatContext.isCritical && Math.random() < 0.15)
          setStatus(enemyStatus, "poisoned", 3); // Poison effect handled elsewhere
        break;

      // --- RARE ---
      case "Venomous Spines":
        if (combatContext.isMelee)
          setStatus(enemyStatus, "poisoned", 5); // Poison effect handled elsewhere
        break;
      case "Light Pulse":
        setStatus(enemyStatus, "blinded", 4);
        gameState.enemyAttackCooldown *= 1.2;
        break;
      case "Venomous Tentacle":
        if (combatContext.isRanged && Math.random() < 0.05)
          gameState.enemyHealth -= 20;
        break;
      case "Crystal Shroud":
        setStatus(playerStatus, "invulnerable", 1);
        break;
      case "Cooldown Link 15":
        gameState.cooldownMultiplier = (gameState.cooldownMultiplier || 1) * 0.85;
        break;
      case "Fierce Territory":
        if (gameState.playerHealth <= 5)
          setStatus(enemyStatus, "fierceDot", 2);
        break;
      case "Angler's Bait":
        setStatus(enemyStatus, "baited", 5);
        break;
      case "Ethereal Flow":
        combatContext.incomingDamage *= 0.75;
        break;
      case "Lightburst":
        setStatus(enemyStatus, "disoriented", 2);
        gameState.enemyAttackCooldown *= 1.25;
        break;
      case "Deflecting Shell":
        if (Math.random() < 0.1)
          gameState.enemyHealth -= combatContext.incomingDamage;
        break;

      // --- EPIC ---
      case "Spirit's Grace":
        setStatus(playerStatus, "spiritGrace", 5);
        break;
      case "Frostbite":
        if (Math.random() < 0.1)
          setStatus(enemyStatus, "frozen", 2);
        break;
      case "Bioluminescent Trail":
        setStatus(enemyStatus, "bioTrailDot", 1);
        gameState.enemyAttackCooldown *= 1.2;
        break;
      case "Abyssal Shield":
        if (!playerStatus.abyssalShield) playerStatus.abyssalShield = 25;
        break;
      case "Clarity of Sight":
        combatContext.damage *= 1.25;
        break;
      case "Cooldown Link 20":
        gameState.cooldownMultiplier = (gameState.cooldownMultiplier || 1) * 0.8;
        break;
      case "Tunneling Strike":
        setStatus(playerStatus, "invulnerable", 2);
        break;
      case "Pincer Crush":
        if (!enemyStatus.stunned) {
          gameState.enemyHealth -= 10;
          setStatus(enemyStatus, "stunned", 3);
        }
        break;
      case "Blinding Flash":
        setStatus(enemyStatus, "stunned", 2);
        break;
      case "Bite of the Abyss":
        setStatus(enemyStatus, "bleed", 1);
        break;

      // --- LEGENDARY ---
      case "Hardened Exoskeleton":
        setStatus(playerStatus, "armorBoost", 5);
        break;
      case "Ventilation Blast":
        gameState.enemyHealth -= 18;
        break;
      case "Evasive Slime":
        if (Math.random() < 0.3) combatContext.enemyAttackMiss = true;
        break;
      case "Calming Aura":
        gameState.playerHealth = Math.max(gameState.playerHealth, 50);
        for (let k in playerStatus) delete playerStatus[k];
        break;
      case "Cooldown Link 25":
        gameState.cooldownMultiplier = (gameState.cooldownMultiplier || 1) * 0.75;
        break;
      case "Shadow Meld":
        setStatus(playerStatus, "invisible", 3);
        setStatus(playerStatus, "immune", 3);
        break;
      case "Ambush Hunter":
        if (combatContext.attackDirection === "behind")
          combatContext.damage *= 1.4;
        break;
      case "Abyssal Glide":
        setStatus(playerStatus, "invulnerable", 3);
        break;
      case "Lacerating Bite":
        gameState.enemyHealth -= 10;
        break;
      case "Adaptive Mimicry":
        if (combatContext.enemySpecial)
          gameState.enemyHealth -= Math.floor(combatContext.enemySpecialValue * 0.5);
        break;

      // --- MYTHIC ---
      case "Venomous Barrage":
        if (!enemyStatus.venomousBarrageUsed) {
          setStatus(enemyStatus, "venomDot", 20);
          enemyStatus.venomousBarrageUsed = true;
        }
        break;
      case "Living Fossil":
        gameState.playerMaxHealth = Math.floor(gameState.playerMaxHealth * 1.5);
        gameState.playerAttackCooldown *= 0.85;
        break;

      // --- SECRET ---
      case "Rejuvenation Cycle":
        if (gameState.playerHealth <= 0 && !playerStatus.revived) {
          gameState.playerHealth = gameState.playerMaxHealth;
          combatContext.damage *= 1.2;
          playerStatus.revived = true;
          setStatus(playerStatus, "rejuvenationBuff", 10);
        }
        break;
    }
  });
}

// --- COMBAT ROUND EXAMPLE ---
function processCombatRound(gameState, playerStatus, enemyStatus, combatContext) {
  // 1. Apply pet perks before attacks
  applyPetPerks(gameState, playerStatus, enemyStatus, combatContext);

  // 2. Handle status effects
  clearExpiredStatus(playerStatus);
  clearExpiredStatus(enemyStatus);

  // 3. Enemy attacks
  let damage = combatContext.incomingDamage || gameState.enemyAttackDamage;
  if (hasStatus(playerStatus, "invulnerable") || hasStatus(playerStatus, "invisible")) {
    combatContext.enemyAttackMiss = true;
    damage = 0;
  } else if (hasStatus(playerStatus, "immune")) {
    damage = 0;
  }
  if (!combatContext.enemyAttackMiss) {
    // Shields
    if (playerStatus.shieldActive) {
      if (playerStatus.shieldActive >= damage) {
        playerStatus.shieldActive -= damage;
        damage = 0;
      } else {
        damage -= playerStatus.shieldActive;
        playerStatus.shieldActive = 0;
      }
    }
    // Abyssal Shield
    if (playerStatus.abyssalShield) {
      if (playerStatus.abyssalShield >= damage) {
        playerStatus.abyssalShield -= damage;
        damage = 0;
      } else {
        damage -= playerStatus.abyssalShield;
        playerStatus.abyssalShield = 0;
      }
    }
    gameState.playerHealth -= damage;
  }

  // 4. Player attacks
  let playerDamage = combatContext.damage || gameState.playerAttackDamage;
  gameState.enemyHealth -= playerDamage;

  // 5. Apply DoTs and buffs/debuffs
  // -- Example: Poison
  if (hasStatus(enemyStatus, "poisoned")) gameState.enemyHealth -= 2; // Or whatever DoT logic you want
  if (hasStatus(enemyStatus, "bleed")) gameState.enemyHealth -= 2;
  if (hasStatus(enemyStatus, "venomDot")) gameState.enemyHealth -= 1;
  if (hasStatus(enemyStatus, "bioTrailDot")) gameState.enemyHealth -= 2;
  if (hasStatus(playerStatus, "spiritGrace")) gameState.playerHealth = Math.min(gameState.playerMaxHealth, gameState.playerHealth + 3);

  // -- Example: fierceDot (enemy takes damage if player is low health)
  if (hasStatus(enemyStatus, "fierceDot")) gameState.enemyHealth -= 2;

  // -- Example: Buff from Living Fossil or Rejuvenation Cycle
  // Add any desired buff processing here

  // 6. Clean up expired or one-time effects if needed

  // 7. Return updated state (or just rely on mutations)
  return { gameState, playerStatus, enemyStatus };
}

export { applyPetPerks, processCombatRound, PETS };
</body>
</html>
