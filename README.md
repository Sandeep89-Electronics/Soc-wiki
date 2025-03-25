# Soc-wiki
Wiki for SoCs
A wiki application for System-on-Chip (SoC) information that can scale across multiple platforms while using open source tools.:

## Tech Stack Recommendation

For your cross-platform SoC wiki application, I suggest using **Electron** with **React** for the initial Windows application. This combination offers several advantages:

1. **Cross-platform compatibility**: The same codebase can be adapted for web and later for mobile
2. **Rich UI libraries**: You can create beautiful interfaces with React component libraries
3. **Data handling**: Excellent support for handling Excel/Word document imports
4. **Open source & free**: The entire stack is free to use
5. **Monetization potential**: Multiple avenues for revenue when you're ready

Let me outline the development approach:

### Windows Application (First Phase)
- **Framework**: Electron (desktop apps using web technologies)
- **Frontend**: React with Material-UI or Chakra UI for polished components
- **Database**: SQLite for local storage, easily migrable later
- **Document Processing**: SheetJS (xlsx) for Excel handling and Mammoth.js for Word docs

### Web Application (Second Phase)
- Convert the Electron app to a web application
- **Backend**: Node.js with Express 
- **Database**: PostgreSQL for robust scalability
- **Hosting**: Deploy on Vercel, Netlify, or similar free tiers initially

### Android Application (Third Phase)
- **Framework**: React Native (reuse much of your React code)
- **Or**: Convert your web app using Capacitor or similar tools

### Monetization Options
1. **Freemium model**: Basic wiki access free, premium features paid
2. **API access**: Charge developers for API access to your SoC database
3. **Sponsored listings**: Feature certain SoCs prominently for a fee
4. **Contextual advertising**: Non-intrusive ads related to semiconductor industry

I'll help you create a Windows application for your SoC wiki, starting with collecting data on various SoC manufacturers like Ambarella, Sigma Star, Novatel, Axera, and Qualcomm.

Let's begin by building a basic Electron application with React that can display and manage SoC data:

// Directory Structure for SoC Wiki Electron App
/*
soc-wiki/
├── package.json
├── main.js
├── preload.js
├── src/
│   ├── App.js
│   ├── components/
│   │   ├── Header.js
│   │   ├── Sidebar.js
│   │   ├── SocCard.js
│   │   ├── SocDetail.js
│   │   ├── SocTable.js
│   │   ├── FileImporter.js
│   │   └── SearchBar.js
│   ├── data/
│   │   └── sampleData.js
│   ├── utils/
│   │   ├── excelParser.js
│   │   ├── wordParser.js
│   │   └── database.js
│   └── index.js
├── index.html
└── assets/
    └── images/
*/

// package.json
{
  "name": "soc-wiki",
  "version": "1.0.0",
  "description": "A wiki application for System-on-Chip information",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "dev": "electron . --dev",
    "build": "electron-builder"
  },
  "dependencies": {
    "electron-store": "^8.1.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "xlsx": "^0.18.5",
    "mammoth": "^1.5.1",
    "@mui/material": "^5.14.0",
    "@emotion/react": "^11.11.0",
    "@emotion/styled": "^11.11.0"
  },
  "devDependencies": {
    "electron": "^25.3.0",
    "electron-builder": "^24.4.0"
  }
}

// main.js (Electron main process)
const { app, BrowserWindow, ipcMain, dialog } = require('electron');
const path = require('path');
const fs = require('fs');
const Store = require('electron-store');

const store = new Store();

function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: false,
      contextIsolation: true
    }
  });

  mainWindow.loadFile('index.html');
  
  // Open DevTools in development mode
  if (process.argv.includes('--dev')) {
    mainWindow.webContents.openDevTools();
  }
}

app.whenReady().then(() => {
  createWindow();
  
  app.on('activate', function () {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit();
});

// Handle file open dialog
ipcMain.handle('open-file-dialog', async () => {
  const { canceled, filePaths } = await dialog.showOpenDialog({
    properties: ['openFile'],
    filters: [
      { name: 'Documents', extensions: ['xlsx', 'xls', 'doc', 'docx'] }
    ]
  });
  
  if (!canceled && filePaths.length > 0) {
    const filePath = filePaths[0];
    const fileData = fs.readFileSync(filePath);
    return {
      path: filePath,
      data: fileData.toString('base64'),
      extension: path.extname(filePath).toLowerCase()
    };
  }
  return null;
});

// preload.js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('electronAPI', {
  openFile: () => ipcRenderer.invoke('open-file-dialog')
});

// index.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SoC Wiki</title>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />
  <style>
    body {
      margin: 0;
      font-family: 'Roboto', sans-serif;
    }
    #root {
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="root"></div>
  <script src="./build/bundle.js"></script>
</body>
</html>

Now, let's implement the React components and sample data for the SoC Wiki application:

// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));

// src/App.js
import React, { useState, useEffect } from 'react';
import { ThemeProvider, createTheme } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';
import Box from '@mui/material/Box';
import Header from './components/Header';
import Sidebar from './components/Sidebar';
import SocTable from './components/SocTable';
import SocDetail from './components/SocDetail';
import { initialSocData } from './data/sampleData';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
    background: {
      default: '#f5f5f5',
    },
  },
});

function App() {
  const [socData, setSocData] = useState(initialSocData);
  const [selectedManufacturer, setSelectedManufacturer] = useState('all');
  const [selectedSoc, setSelectedSoc] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');

  // Filter SoCs based on selected manufacturer and search term
  const filteredSocs = socData.filter(soc => {
    const matchesManufacturer = selectedManufacturer === 'all' || soc.manufacturer === selectedManufacturer;
    const matchesSearch = soc.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                          soc.description.toLowerCase().includes(searchTerm.toLowerCase());
    return matchesManufacturer && matchesSearch;
  });

  // Get unique manufacturers for sidebar
  const manufacturers = ['all', ...new Set(socData.map(soc => soc.manufacturer))];

  const handleImportFile = async (fileData) => {
    // This would be implemented with actual file parsing logic
    console.log('Importing file:', fileData);
    // For now, we'll just use our sample data
    // In a real app, you'd parse the Excel/Word file and update socData
  };

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Box sx={{ display: 'flex', height: '100vh' }}>
        <Header onSearch={setSearchTerm} onImport={handleImportFile} />
        <Sidebar 
          manufacturers={manufacturers} 
          selectedManufacturer={selectedManufacturer}
          onSelectManufacturer={setSelectedManufacturer}
        />
        <Box sx={{ flexGrow: 1, p: 3, mt: 8, display: 'flex' }}>
          <Box sx={{ width: selectedSoc ? '60%' : '100%', pr: 2 }}>
            <SocTable 
              socs={filteredSocs} 
              onSelectSoc={setSelectedSoc} 
              selectedSoc={selectedSoc}
            />
          </Box>
          {selectedSoc && (
            <Box sx={{ width: '40%' }}>
              <SocDetail soc={selectedSoc} />
            </Box>
          )}
        </Box>
      </Box>
    </ThemeProvider>
  );
}

export default App;

// src/components/Header.js
import React from 'react';
import AppBar from '@mui/material/AppBar';
import Toolbar from '@mui/material/Toolbar';
import Typography from '@mui/material/Typography';
import Button from '@mui/material/Button';
import Box from '@mui/material/Box';
import SearchBar from './SearchBar';

function Header({ onSearch, onImport }) {
  const handleFileImport = async () => {
    try {
      const fileData = await window.electronAPI.openFile();
      if (fileData) {
        onImport(fileData);
      }
    } catch (error) {
      console.error('Error importing file:', error);
    }
  };

  return (
    <AppBar position="fixed">
      <Toolbar>
        <Typography variant="h6" component="div" sx={{ flexGrow: 1 }}>
          SoC Wiki
        </Typography>
        <Box sx={{ display: 'flex', alignItems: 'center' }}>
          <SearchBar onSearch={onSearch} />
          <Button 
            color="inherit" 
            onClick={handleFileImport}
            sx={{ ml: 2 }}
          >
            Import Data
          </Button>
        </Box>
      </Toolbar>
    </AppBar>
  );
}

export default Header;

// src/components/SearchBar.js
import React, { useState } from 'react';
import TextField from '@mui/material/TextField';
import InputAdornment from '@mui/material/InputAdornment';
import SearchIcon from '@mui/icons-material/Search';

function SearchBar({ onSearch }) {
  const [searchTerm, setSearchTerm] = useState('');

  const handleChange = (e) => {
    const value = e.target.value;
    setSearchTerm(value);
    onSearch(value);
  };

  return (
    <TextField
      placeholder="Search SoCs..."
      variant="outlined"
      size="small"
      value={searchTerm}
      onChange={handleChange}
      InputProps={{
        startAdornment: (
          <InputAdornment position="start">
            <SearchIcon />
          </InputAdornment>
        ),
      }}
      sx={{ 
        backgroundColor: 'rgba(255, 255, 255, 0.15)',
        borderRadius: 1,
        '& .MuiOutlinedInput-root': {
          '& fieldset': {
            borderColor: 'transparent',
          },
          '&:hover fieldset': {
            borderColor: 'transparent',
          },
          '&.Mui-focused fieldset': {
            borderColor: 'transparent',
          },
        },
        input: {
          color: 'white',
        }
      }}
    />
  );
}

export default SearchBar;

// src/components/Sidebar.js
import React from 'react';
import Drawer from '@mui/material/Drawer';
import List from '@mui/material/List';
import ListItem from '@mui/material/ListItem';
import ListItemButton from '@mui/material/ListItemButton';
import ListItemText from '@mui/material/ListItemText';
import Typography from '@mui/material/Typography';
import Box from '@mui/material/Box';

function Sidebar({ manufacturers, selectedManufacturer, onSelectManufacturer }) {
  return (
    <Drawer
      variant="permanent"
      sx={{
        width: 240,
        flexShrink: 0,
        '& .MuiDrawer-paper': {
          width: 240,
          boxSizing: 'border-box',
          mt: 8,
        },
      }}
    >
      <Box sx={{ overflow: 'auto', p: 2 }}>
        <Typography variant="h6" sx={{ mb: 2 }}>
          Manufacturers
        </Typography>
        <List>
          {manufacturers.map((manufacturer) => (
            <ListItem key={manufacturer} disablePadding>
              <ListItemButton
                selected={selectedManufacturer === manufacturer}
                onClick={() => onSelectManufacturer(manufacturer)}
              >
                <ListItemText 
                  primary={manufacturer === 'all' ? 'All Manufacturers' : manufacturer} 
                />
              </ListItemButton>
            </ListItem>
          ))}
        </List>
      </Box>
    </Drawer>
  );
}

export default Sidebar;

// src/components/SocTable.js
import React from 'react';
import Table from '@mui/material/Table';
import TableBody from '@mui/material/TableBody';
import TableCell from '@mui/material/TableCell';
import TableContainer from '@mui/material/TableContainer';
import TableHead from '@mui/material/TableHead';
import TableRow from '@mui/material/TableRow';
import Paper from '@mui/material/Paper';
import Typography from '@mui/material/Typography';

function SocTable({ socs, onSelectSoc, selectedSoc }) {
  return (
    <Paper sx={{ width: '100%', overflow: 'hidden' }}>
      <Typography variant="h6" sx={{ p: 2 }}>
        SoC Database
      </Typography>
      <TableContainer sx={{ maxHeight: 'calc(100vh - 180px)' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              <TableCell>Name</TableCell>
              <TableCell>Manufacturer</TableCell>
              <TableCell>CPU</TableCell>
              <TableCell>GPU</TableCell>
              <TableCell>Process</TableCell>
              <TableCell>Release Year</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {socs.map((soc) => (
              <TableRow 
                key={soc.id} 
                hover 
                onClick={() => onSelectSoc(soc)}
                selected={selectedSoc && selectedSoc.id === soc.id}
                sx={{ cursor: 'pointer' }}
              >
                <TableCell>{soc.name}</TableCell>
                <TableCell>{soc.manufacturer}</TableCell>
                <TableCell>{soc.cpu}</TableCell>
                <TableCell>{soc.gpu}</TableCell>
                <TableCell>{soc.process}</TableCell>
                <TableCell>{soc.releaseYear}</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Paper>
  );
}

export default SocTable;

// src/components/SocDetail.js
import React from

Let me continue the development by completing the database util and adding the file importer component. I'll also add a function to handle the file import process.

// src/data/sampleData.js
export const initialSocData = [
  {
    id: '1',
    name: 'Snapdragon 8 Gen 3',
    manufacturer: 'Qualcomm',
    cpu: 'Kryo 780 (1x3.3 GHz + 3x3.2 GHz + 2x3.0 GHz + 2x2.3 GHz)',
    gpu: 'Adreno 750',
    process: '4nm TSMC',
    memory: 'LPDDR5X',
    releaseYear: 2023,
    description: 'Qualcomm\'s flagship SoC for premium smartphones with powerful AI capabilities and gaming performance.',
    features: ['Wi-Fi 7', '5G', 'AI Engine', '8K Video', 'Bluetooth 5.3'],
    benchmarks: 'AnTuTu: 1,500,000+, Geekbench 5: Single-Core: 2000, Multi-Core: 6500'
  },
  {
    id: '2',
    name: 'CV52S',
    manufacturer: 'Ambarella',
    cpu: 'Quad-core Arm Cortex-A76',
    gpu: 'Integrated GPU',
    process: '5nm',
    memory: 'LPDDR4X/5',
    releaseYear: 2023,
    description: 'Ambarella\'s advanced SoC designed for automotive, security and robotics applications with advanced computer vision capabilities.',
    features: ['CVflow Architecture', '8K Video', 'Neural Network Processor', 'Advanced ISP', 'HDR Processing'],
    benchmarks: 'AI Performance: 20 TOPS'
  },
  {
    id: '3',
    name: 'SIP6501',
    manufacturer: 'Sigma Star',
    cpu: 'Dual-core ARM Cortex-A53',
    gpu: 'Mali-G52',
    process: '12nm',
    memory: 'LPDDR4',
    releaseYear: 2022,
    description: 'Cost-effective SoC designed for IP cameras and security applications with good power efficiency.',
    features: ['H.265 Codec', 'Smart Analytics', 'LDC', 'WDR', 'Advanced ISP'],
    benchmarks: 'Power Consumption: <1.5W'
  },
  {
    id: '4',
    name: 'MTK8395',
    manufacturer: 'Novatel',
    cpu: 'Octa-core (2x2.2 GHz + 6x1.8 GHz)',
    gpu: 'PowerVR GE8320',
    process: '14nm',
    memory: 'LPDDR4X',
    releaseYear: 2021,
    description: 'Mid-range SoC for IoT and connectivity solutions with integrated LTE modem.',
    features: ['LTE Cat-4', 'GPS/GLONASS', 'Wi-Fi 6', 'Bluetooth 5.1', 'eMTC'],
    benchmarks: 'N/A'
  },
  {
    id: '5',
    name: 'AX620A',
    manufacturer: 'Axera',
    cpu: 'Quad-core Arm Cortex-A53',
    gpu: 'Custom NPU',
    process: '12nm',
    memory: 'DDR4',
    releaseYear: 2022,
    description: 'AI-focused SoC for intelligent video analysis and edge computing applications.',
    features: ['5 TOPS NPU', '4K Encoding', 'ISP', 'Multi-Object Tracking', 'Face Recognition'],
    benchmarks: 'AI Processing: 5 TOPS'
  },
  {
    id: '6',
    name: 'MediaTek Dimensity 9300',
    manufacturer: 'MediaTek',
    cpu: 'Octa-core (4x Cortex-X4 + 4x Cortex-A720)',
    gpu: 'Immortalis-G720',
    process: '4nm TSMC',
    memory: 'LPDDR5X',
    releaseYear: 2023,
    description: 'MediaTek\'s flagship SoC with all big-core design for premium Android smartphones.',
    features: ['5G', 'Wi-Fi 7', 'Ray Tracing', 'AV1 Decode', 'MiraVision 850'],
    benchmarks: 'AnTuTu: 1,300,000+, Geekbench 5: Single-Core: 1900, Multi-Core: 6300'
  },
  {
    id: '7',
    name: 'CV3',
    manufacturer: 'Ambarella',
    cpu: 'Multi-core ARM Cortex-A78AE',
    gpu: 'Integrated GPU',
    process: '5nm',
    memory: 'LPDDR5',
    releaseYear: 2022,
    description: 'Advanced automotive SoC designed for ADAS and autonomous driving applications.',
    features: ['CVflow Architecture', '4K Stereo Processing', 'Radar Processing', 'Camera Fusion', '100 TOPS AI'],
    benchmarks: 'AI Performance: 100 TOPS'
  },
  {
    id: '8',
    name: 'SIP7501',
    manufacturer: 'Sigma Star',
    cpu: 'Quad-core ARM Cortex-A55',
    gpu: 'Mali-G57',
    process: '10nm',
    memory: 'LPDDR4X',
    releaseYear: 2023,
    description: 'High-performance SoC for advanced security cameras and edge AI applications.',
    features: ['H.266 Support', 'VISP 3.0', 'AI Engine', '4K@60fps', 'Multiple Stream Processing'],
    benchmarks: 'AI Performance: 3 TOPS'
  }
];

// src/utils/excelParser.js
import * as XLSX from 'xlsx';

/**
 * Parse Excel file and extract SoC data
 * @param {ArrayBuffer} fileData - The file data as ArrayBuffer
 * @returns {Array} Array of SoC objects
 */
export function parseExcelFile(fileData) {
  // Convert base64 to ArrayBuffer if needed
  let data = fileData;
  if (typeof fileData === 'string') {
    const binaryString = window.atob(fileData);
    const len = binaryString.length;
    data = new ArrayBuffer(len);
    const view = new Uint8Array(data);
    for (let i = 0; i < len; i++) {
      view[i] = binaryString.charCodeAt(i);
    }
  }

  // Read the Excel file
  const workbook = XLSX.read(data, { type: 'array' });
  const firstSheetName = workbook.SheetNames[0];
  const worksheet = workbook.Sheets[firstSheetName];
  
  // Convert to JSON
  const jsonData = XLSX.utils.sheet_to_json(worksheet);
  
  // Map the Excel data to our SoC data structure
  return jsonData.map((row, index) => {
    // Assuming the Excel has columns that match our data structure
    // Adjust field mapping as needed based on your Excel structure
    return {
      id: String(index + 1),
      name: row.Name || row.name || '',
      manufacturer: row.Manufacturer || row.manufacturer || '',
      cpu: row.CPU || row.cpu || '',
      gpu: row.GPU || row.gpu || '',
      process: row.Process || row.process || '',
      memory: row.Memory || row.memory || '',
      releaseYear: row.ReleaseYear || row.releaseYear || new Date().getFullYear(),
      description: row.Description || row.description || '',
      features: row.Features ? 
        (typeof row.Features === 'string' ? row.Features.split(',').map(f => f.trim()) : []) : 
        [],
      benchmarks: row.Benchmarks || row.benchmarks || ''
    };
  });
}

// src/utils/wordParser.js
import mammoth from 'mammoth';

/**
 * Parse Word document and extract SoC data
 * This is a simplified version that assumes structured Word docs
 * @param {ArrayBuffer} fileData - The file data as ArrayBuffer
 * @returns {Promise<Array>} Promise resolving to array of SoC objects
 */
export async function parseWordFile(fileData) {
  // Convert base64 to ArrayBuffer if needed
  let data = fileData;
  if (typeof fileData === 'string') {
    const binaryString = window.atob(fileData);
    const len = binaryString.length;
    data = new ArrayBuffer(len);
    const view = new Uint8Array(data);
    for (let i = 0; i < len; i++) {
      view[i] = binaryString.charCodeAt(i);
    }
  }

  // Extract text from Word document
  const result = await mammoth.extractRawText({ arrayBuffer: data });
  const text = result.value;
  
  // Simple parser for structured Word docs
  // This is a basic implementation - would need to be adjusted based on your document structure
  const socEntries = text.split(/(?=Name:)/);
  
  return socEntries.filter(entry => entry.trim().length > 0).map((entry, index) => {
    const lines = entry.split('\n').map(line => line.trim());
    const socData = {
      id: String(index + 1),
      features: []
    };
    
    let inFeatures = false;
    
    for (const line of lines) {
      if (line.startsWith('Name:')) {
        socData.name = line.replace('Name:', '').trim();
      } else if (line.startsWith('Manufacturer:')) {
        socData.manufacturer = line.replace('Manufacturer:', '').trim();
      } else if (line.startsWith('CPU:')) {
        socData.cpu = line.replace('CPU:', '').trim();
      } else if (line.startsWith('GPU:')) {
        socData.gpu = line.replace('GPU:', '').trim();
      } else if (line.startsWith('Process:')) {
        socData.process = line.replace('Process:', '').trim();
      } else if (line.startsWith('Memory:')) {
        socData.memory = line.replace('Memory:', '').trim();
      } else if (line.startsWith('Release Year:')) {
        socData.releaseYear = parseInt(line.replace('Release Year:', '').trim()) || new Date().getFullYear();
      } else if (line.startsWith('Description:')) {
        socData.description = line.replace('Description:', '').trim();
      } else if (line.startsWith('Benchmarks:')) {
        socData.benchmarks = line.replace('Benchmarks:', '').trim();
        inFeatures = false;
      } else if (line.startsWith('Features:')) {
        inFeatures = true;
      } else if (inFeatures && line.trim()) {
        // Collect features until we hit the benchmarks section
        socData.features.push(line.replace('-', '').trim());
      }
    }
    
    return socData;
  });
}

// src/utils/database.js
import Store from 'electron-store';

class SocDatabase {
  constructor() {
    this.store = new Store();
    
    // Initialize with default data if empty
    if (!this.store.has('socs')) {
      this.store.set('socs', []);
    }
  }
  
  // Get all SoCs
  getAllSocs() {
    return this.store.get('socs');
  }
  
  // Add SoCs (either single or array)
  addSocs(socData) {
    const currentSocs = this.getAllSocs();
    const newSocs = Array.isArray(socData) ? socData : [socData];
    
    // Generate IDs for new SoCs if they don't have one
    newSocs.forEach((soc, idx) => {
      if (!soc.id) {
        soc.id = String(Date.now() + idx);
      }
    });
    
    this.store.set('socs', [...currentSocs, ...newSocs]);
    return newSocs;
  }
  
  // Update a SoC
  updateSoc(id, socData) {
    const currentSocs = this.getAllSocs();
    const updatedSocs = currentSocs.map(soc => 
      soc.id === id ? { ...soc, ...socData } : soc
    );
    
    this.store.set('socs', updatedSocs);
    return updatedSocs.find(soc => soc.id === id);
  }
  
  // Delete a SoC
  deleteSoc(id) {
    const currentSocs = this.getAllSocs();
    const updatedSocs = currentSocs.filter(soc => soc.id !== id);
    
    this.store.set('socs', updatedSocs);
    return id;
  }
  
  // Search SoCs
  searchSocs(term) {
    const currentSocs = this.getAllSocs();
    if (!term) return currentSocs;
    
    const searchTerm = term.toLowerCase();
    return currentSocs.filter(soc => 
      soc.name.toLowerCase().includes(searchTerm) ||
      soc.manufacturer.toLowerCase().includes(searchTerm) ||
      soc.description.toLowerCase().includes(searchTerm) ||
      soc.cpu.toLowerCase().includes(searchTerm) ||
      soc.gpu.toLowerCase().includes(searchTerm)
    );
  }
  
  // Clear all data
  clearAll() {
    this.store.set('socs', []);
  }
}

export default new SocDatabase();

Now, let's create the FileImporter component and add the functionality to handle file imports:

// src/components/FileImporter.js
import React, { useState } from 'react';
import Dialog from '@mui/material/Dialog';
import DialogActions from '@mui/material/DialogActions';
import DialogContent from '@mui/material/DialogContent';
import DialogContentText from '@mui/material/DialogContentText';
import DialogTitle from '@mui/material/DialogTitle';
import Button from '@mui/material/Button';
import Box from '@mui/material/Box';
import Typography from '@mui/material/Typography';
import CircularProgress from '@mui/material/CircularProgress';
import { parseExcelFile } from '../utils/excelParser';
import { parseWordFile } from '../utils/wordParser';

function FileImporter({ open, onClose, onImportComplete }) {
  const [isLoading, setIsLoading] = useState(false);
  const [fileInfo, setFileInfo] = useState(null);
  const [error, setError] = useState(null);
  
  const handleFileSelect = async () => {
    try {
      setIsLoading(true);
      setError(null);
      
      const fileData = await window.electronAPI.openFile();
      
      if (fileData) {
        setFileInfo({
          name: fileData.path.split('\\').pop().split('/').pop(),
          extension: fileData.extension,
          path: fileData.path
        });
      }
      
      setIsLoading(false);
    } catch (error) {
      setError('Error selecting file: ' + error.message);
      setIsLoading(false);
    }
  };
  
  const handleImport = async () => {
    if (!fileInfo) return;
    
    try {
      setIsLoading(true);
      setError(null);
      
      const fileData = await window.electronAPI.openFile();
      let parsedData = [];
      
      if (fileData.extension === '.xlsx' || fileData.extension === '.xls') {
        parsedData = parseExcelFile(fileData.data);
      } else if (fileData.extension === '.docx' || fileData.extension === '.doc') {
        parsedData = await parseWordFile(fileData.data);
      } else {
        throw new Error('Unsupported file format');
      }
      
      // Pass the parsed data back to parent component
      onImportComplete(parsedData);
      setIsLoading(false);
      onClose();
      
    } catch (error) {
      setError('Error importing file: ' + error.message);
      setIsLoading(false);
    }
  };
  
  return (
    <Dialog
      open={open}
      onClose={onClose}
      aria-labelledby="file-importer-dialog-title"
    >
      <DialogTitle id="file-importer-dialog-title">
        Import SoC Data
      </DialogTitle>
      <DialogContent>
        <DialogContentText>
          Select an Excel (.xlsx, .xls) or Word (.docx, .doc) file containing SoC data.
          The file should have columns or structured text matching the SoC information format.
        </DialogContentText>
        
        <Box sx={{ mt: 3, mb: 2, display: 'flex', flexDirection: 'column', alignItems: 'center' }}>
          {!fileInfo && !isLoading && (
            <Button
              variant="contained"
              onClick={handleFileSelect}
              sx={{ mb: 2 }}
            >
              Select File
            </Button>
          )}
          
          {isLoading && (
            <CircularProgress sx={{ mb: 2 }} />
          )}
          
          {fileInfo && !isLoading && (
            <Box sx={{ textAlign: 'center', mb: 2 }}>
              <Typography variant="body1">
                Selected file: {fileInfo.name}
              </Typography>
              <Button
                variant="text"
                onClick={handleFileSelect}
                size="small"
              >
                Change File
              </Button>
            </Box>
          )}
          
          {error && (
            <Typography color="error" variant="body2" sx={{ mt: 1 }}>
              {error}
            </Typography>
          )}
        </Box>
      </DialogContent>
      <DialogActions>
        <Button onClick={onClose} color="primary">
          Cancel
        </Button>
        <Button
          onClick={handleImport}
          color="primary"
          variant="contained"
          disabled={!fileInfo || isLoading}
        >
          Import
        </Button>
      </DialogActions>
    </Dialog>
  );
}

export default FileImporter;

Let's update the main App component to handle the file import dialog and update the data properly:

// src/App.js (Updated)
import React, { useState, useEffect } from 'react';
import { ThemeProvider, createTheme } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';
import Box from '@mui/material/Box';
import Snackbar from '@mui/material/Snackbar';
import Alert from '@mui/material/Alert';
import Header from './components/Header';
import Sidebar from './components/Sidebar';
import SocTable from './components/SocTable';
import SocDetail from './components/SocDetail';
import FileImporter from './components/FileImporter';
import { initialSocData } from './data/sampleData';
import socDatabase from './utils/database';
import { parseExcelFile } from './utils/excelParser';
import { parseWordFile } from './utils/wordParser';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
    background: {
      default: '#f5f5f5',
    },
  },
});

function App() {
  const [socData, setSocData] = useState(initialSocData);
  const [selectedManufacturer, setSelectedManufacturer] = useState('all');
  const [selectedSoc, setSelectedSoc] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [fileImporterOpen, setFileImporterOpen] = useState(false);
  const [notification, setNotification] = useState({ open: false, message: '', severity: 'info' });

  // Load data from the database on startup
  useEffect(() => {
    const loadData = () => {
      const storedData = socDatabase.getAllSocs();
      
      // If there's no data in the database, initialize with sample data
      if (storedData.length === 0) {
        socDatabase.addSocs(initialSocData);
        setSocData(initialSocData);
      } else {
        setSocData(storedData);
      }
    };
    
    loadData();
  }, []);

  // Filter SoCs based on selected manufacturer and search term
  const filteredSocs = socData.filter(soc => {
    const matchesManufacturer = selectedManufacturer === 'all' || soc.manufacturer === selectedManufacturer;
    const matchesSearch = searchTerm === '' || 
                          soc.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                          soc.description.toLowerCase().includes(searchTerm.toLowerCase()) ||
                          soc.cpu.toLowerCase().includes(searchTerm.toLowerCase()) ||
                          soc.gpu.toLowerCase().includes(searchTerm.toLowerCase());
    return matchesManufacturer && matchesSearch;
  });

  // Get unique manufacturers for sidebar
  const manufacturers = ['all', ...Array.from(new Set(socData.map(soc => soc.manufacturer)))];

  const handleImportFile = async (fileData) => {
    try {
      let parsedData = [];
      
      if (fileData.extension === '.xlsx' || fileData.extension === '.xls') {
        parsedData = parseExcelFile(fileData.data);
      } else if (fileData.extension === '.docx' || fileData.extension === '.doc') {
        parsedData = await parseWordFile(fileData.data);
      } else {
        throw new Error('Unsupported file format');
      }
      
      // Add the parsed data to the database
      socDatabase.addSocs(parsedData);
      
      // Update the local state
      const updatedData = socDatabase.getAllSocs();
      setSocData(updatedData);
      
      setNotification({
        open: true,
        message: `Successfully imported ${parsedData.length} SoCs`,
        severity: 'success'
      });
      
    } catch (error) {
      console.error('Error importing file:', error);
      setNotification({
        open: true,
        message: `Error importing file: ${error.message}`,
        severity: 'error'
      });
    }
  };

  const handleFileImporterOpen = () => {
    setFileImporterOpen(true);
  };

  const handleFileImporterClose = () => {
    setFileImporterOpen(false);
  };

  const handleImportComplete = (parsedData) => {
    // Add the parsed data to the database
    socDatabase.addSocs(parsedData);
    
    // Update the local state
    const updatedData = socDatabase.getAllSocs();
    setSocData(updatedData);
    
    setNotification({
      open: true,
      message: `Successfully imported ${parsedData.length} SoCs`,
      severity: 'success'
    });
  };

  const handleCloseNotification = (event, reason) => {
    if (reason === 'clickaway') {
      return;
    }
    setNotification({ ...notification, open: false });
  };

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Box sx={{ display: 'flex', height: '100vh' }}>
        <Header 
          onSearch={setSearchTerm} 
          onImport={handleFileImporterOpen} 
        />
        <Sidebar 
          manufacturers={manufacturers} 
          selectedManufacturer={selectedManufacturer}
          onSelectManufacturer={setSelectedManufacturer}
        />
        <Box sx={{ flexGrow: 1, p: 3, mt: 8, display: 'flex' }}>
          <Box sx={{ width: selectedSoc ? '60%' : '100%', pr: 2 }}>
            <SocTable 
              socs={filteredSocs} 
              onSelectSoc={setSelectedSoc} 
              selectedSoc={selectedSoc}
            />
          </Box>
          {selectedSoc && (
            <Box sx={{ width: '40%' }}>
              <SocDetail soc={selectedSoc} />
            </Box>
          )}
        </Box>
        
        <FileImporter 
          open={fileImporterOpen} 
          onClose={handleFileImporterClose}
          onImportComplete={handleImportComplete}
        />
        
        <Snackbar 
          open={notification.open} 
          autoHideDuration={6000} 
          onClose={handleCloseNotification}
        >
          <Alert 
            onClose={handleCloseNotification} 
            severity={notification.severity} 
            sx={{ width: '100%' }}
          >
            {notification.message}
          </Alert>
        </Snackbar>
      </Box>
    </ThemeProvider>
  );
}

export default App;

Now, let's update the main.js file to include IPC handlers for file operations:

// main.js (Updated)
const { app, BrowserWindow, ipcMain, dialog } = require('electron');
const path = require('path');
const fs = require('fs');
const Store = require('electron-store');

// Initialize the store
const store = new Store();

// Create the browser window
function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: false,
      contextIsolation: true
    }
  });

  // Loa
