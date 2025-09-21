# DSpace Angular Frontend Documentation

## Table of Contents
1. Introduction
2. Getting Started
3. Application Structure
4. Authentication
5. Case Files Search Implementation
6. API Endpoints
7. Internationalization

## Introduction

This documentation provides a comprehensive guide to the DSpace Angular frontend application. It is designed to help new developers understand the application structure, workflow, and implementation details to facilitate efficient onboarding and development.

## Getting Started

### Prerequisites
- Node.js (recommended version: 14.x or higher)
- npm or yarn package manager

### Installation and Setup
1. Navigate to the frontend root directory:
   cd /path/to/frontend

2. Install dependencies:
   npm install
   or
   yarn install

3. Start the development server:
   npm start
   or
   yarn start

4. The application will be available at http://localhost:4000 (default port)

## Application Structure

### Routing Configuration
The application uses Angular's routing system to manage navigation. The main routes are defined in:
- src/app/app-routes.ts

This file contains the primary route definitions and lazy-loaded modules. For example:
{ 
  path: 'login', 
  loadChildren: () => import('./login-page/login-page-routes').then((m) => m.ROUTES)
}

Each feature module typically has its own routes file (e.g., login-page-routes.ts) that defines the specific routes for that feature.

### Component Structure
Components follow the standard Angular structure:
- .component.ts - Component logic and functionality
- .component.html - Component template (UI structure)
- .component.scss - Component-specific styles
- .module.ts - Module definition, imports, and declarations

### Module Registration
New modules must be registered in src/app/root.module.ts to be available throughout the application.

## Authentication

### Login Implementation
The login functionality is implemented in:
- login-page.component.ts - Contains the login logic
- login-page.component.html - Contains the login UI structure

The login page is accessible at the /login route.

### Authentication Guard
The application implements route protection using the authenticatedGuard. This guard is applied to routes that require authentication:

{
  path: 'some-protected-route',
  component: SomeComponent,
  canActivate: [authenticatedGuard]
}

If a user attempts to access a protected route without being logged in, they will be redirected to the login page.

## Case Files Search Implementation

### Browse Configuration
The browse functionality for case files is configured in the DSpace backend configuration (dspace.cfg):

browse.comcol.by.dc_case_number
browse.comcol.by.dc_case_type
browse.comcol.by.dc_case_year

### UI Implementation
The search functionality for case files is implemented through a custom component:

1. Component Files:
   - search-page.component.html - UI interface with input fields and search results
   - search-page.component.scss - Styling for the search page
   - search-page.component.ts - Search functionality implementation
   - search-page.module.ts - Module imports (NgModule, CommonModule, FormsModule, HttpClientModule, RouterModule)

2. Route Configuration:
   The search component is connected to the browse route in app-routes.ts:
   {
     path: 'browse',
     canActivate: [endUserAgreementCurrentUserGuard],
     children: [
       {
         path: 'title',
         component: SearchPageComponent,
         data: { enableRSS: true },
         canActivate: [endUserAgreementCurrentUserGuard]
       }
     ]
   }

3. Service Implementation:
   - Created service files (.service.ts) to handle API calls
   - Services use Angular's HttpClient to make requests to the backend

4. Search Flow:
   - User selects search parameters (case type, number, year)
   - Application calls the search API with the selected parameters
   - Results are displayed in the UI
   - Clicking on a result redirects to the item viewer page

## API Endpoints

### Search API
Endpoint for searching case files with filters:
http://localhost:8080/server/api/discover/search/objects?sort=dc.title,ASC&size=10&f.dc_case_number={number},equals&f.dc_case_type={type},equals&f.dc_case_year={year},equals

Parameters:
- sort - Sorting parameter (e.g., dc.title,ASC)
- size - Number of results to return
- f.dc_case_number - Filter by case number
- f.dc_case_type - Filter by case type
- f.dc_case_year - Filter by case year

### Facets API
Endpoint for retrieving available case types for dropdown:
http://localhost:8080/server/api/discover/facets/dc_case_type?size=1000

Parameters:
- size - Number of facet values to return

## Internationalization

### Label Configuration
User-friendly labels for browse options are defined in the internationalization file:
- src/assets/i18n/en.json5

Example configuration:
"browse.comcol.by.dc_case_number": "By Case number",
"browse.comcol.by.dc_case_type": "By Case type",
"browse.comcol.by.dc_case_year": "By Case year"

This mapping transforms the technical identifiers from the backend into user-friendly labels in the UI.
