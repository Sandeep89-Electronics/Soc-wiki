// src/index.js
import React from 'react';
import Card from '@mui/material/Card';
import CardContent from '@mui/material/CardContent';
import Typography from '@mui/material/Typography';
import Grid from '@mui/material/Grid';
import Chip from '@mui/material/Chip';
import Box from '@mui/material/Box';

function SocDetail({ soc }) {
  return (
    <Card sx={{ height: '100%' }}>
      <CardContent>
        <Typography variant="h5" component="div" gutterBottom>
          {soc.name}
        </Typography>
        <Typography variant="subtitle1" color="text.secondary" gutterBottom>
          {soc.manufacturer} • {soc.releaseYear}
        </Typography>
        
        <Typography variant="body2" color="text.secondary" paragraph>
          {soc.description}
        </Typography>
        
        <Grid container spacing={2}>
          <Grid item xs={6}>
            <Typography variant="subtitle2">CPU</Typography>
            <Typography variant="body2" paragraph>{soc.cpu}</Typography>
          </Grid>
          <Grid item xs={6}>
            <Typography variant="subtitle2">GPU</Typography>
            <Typography variant="body2" paragraph>{soc.gpu}</Typography>
          </Grid>
          <Grid item xs={6}>
            <Typography variant="subtitle2">Process</Typography>
            <Typography variant="body2" paragraph>{soc.process}</Typography>
          </Grid>
          <Grid item xs={6}>
            <Typography variant="subtitle2">Memory</Typography>
            <Typography variant="body2" paragraph>{soc.memory}</Typography>
          </Grid>
          <Grid item xs={12}>
            <Typography variant="subtitle2">Key Features</Typography>
            <Box sx={{ mt: 1 }}>
              {soc.features.map((feature, index) => (
                <Chip 
                  key={index} 
                  label={feature} 
                  size="small" 
                  sx={{ mr: 1, mb: 1 }} 
                />
              ))}
            </Box>
          </Grid>
          {soc.benchmarks && (
            <Grid item xs={12}>
              <Typography variant="subtitle2">Benchmarks</Typography>
              <Typography variant="body2" paragraph>{soc.benchmarks}</Typography>
            </Grid>
          )}
        </Grid>
      </CardContent>
    </Card>
  );
}

export default SocDetail;'react';
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
