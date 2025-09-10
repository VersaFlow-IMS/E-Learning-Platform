# eLearning Management System - System Overview

## Project Introduction

The eLearning Management System (LMS) is a comprehensive educational platform designed to facilitate online learning for teachers, students, parents, administrators, and support developers. The system follows a phased delivery approach (V1-V4) to ensure progressive functionality and continuous improvement.

## Technology Stack

### Backend
- **Framework**: .NET 6/7 (C#)
- **Database**: SQL Server
- **Authentication**: ASP.NET Core Identity
- **API**: RESTful Web API
- **ORM**: Entity Framework Core

### Frontend
- **Framework**: Angular 15+
- **UI Library**: Angular Material / Bootstrap
- **State Management**: NgRx (recommended for complex state)
- **HTTP Client**: Angular HttpClient
- **Authentication**: JWT tokens with Angular Guards

### Infrastructure
- **Development Environment**: Two-tier (Production & Stage/Test)
- **Future Cloud**: AWS (V4)
- **Containerization**: Docker (V4)
- **Mobile**: Cross-platform app (V3)

## System Architecture

### High-Level Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Angular SPA   │    │   .NET Web API  │    │   SQL Server    │
│   (Frontend)    │◄──►│   (Backend)     │◄──►│   Database      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Database Strategy
- **Production Database**: Live system data
- **Stage/Test Database**: Clone of production for testing
- **Data Migration**: Admin can copy data from Prod → Stage/Test

## User Roles & Permissions

### Administrator
- **Primary Responsibilities**: System management and teacher account administration
- **Access Level**: Full system access including production database
- **Key Features**:
  - Create/manage teacher accounts
  - Contract management (start/end dates, renewals)
  - Data migration between environments

### Support Developer
- **Primary Responsibilities**: System testing and development support
- **Access Level**: Stage/Test environment only
- **Key Features**:
  - Full testing capabilities
  - No production data access (except cloned data)

### Teacher
- **Primary Responsibilities**: Class and content management
- **Access Level**: Class and student management within assigned scope
- **Key Features**:
  - Profile management (personal info, images, logos)
  - Class creation and management
  - Student assignment and transfers
  - Content upload and visibility control
  - QR code generation for attendance
  - Analytics dashboard

### Student
- **Primary Responsibilities**: Learning and content consumption
- **Access Level**: Assigned classes and content only
- **Key Features**:
  - Class participation
  - Content access (PDFs, videos, documents)
  - Performance tracking
  - Dashboard with insights

### Parent (V3)
- **Primary Responsibilities**: Child progress monitoring
- **Access Level**: Child-specific data only
- **Key Features**:
  - Multi-child account support
  - Progress and performance tracking
  - Attendance monitoring

## Version Roadmap

### Version 1 (Core System) - Current Focus
**Objective**: Establish foundational LMS functionality

**Key Deliverables**:
- User authentication system
- Multi-role access control
- Teacher profile and class management
- Student enrollment and content access
- File management system
- Dual database environment (Prod/Stage)
- Basic dashboards

**Success Criteria**:
- Teachers can create classes and upload content
- Students can access assigned materials
- Admin can manage teacher accounts
- Support developers can test safely

### Version 2 (Learning Activities)
**Objective**: Enhance educational features

**Additions**:
- Assignment management with deadlines
- Quiz creation and management
- Calendar/agenda system
- Advanced analytics

### Version 3 (Engagement & Parents)
**Objective**: Expand platform reach and engagement

**Additions**:
- Mobile applications (iOS/Android)
- Video conferencing integration
- QR-based attendance system
- Parent portal

### Version 4 (Cloud & Scalability)
**Objective**: Enterprise-grade scalability

**Additions**:
- AWS cloud migration
- Docker containerization
- Performance optimization
- Advanced analytics

## Core Business Rules

### Student Management
- **Uniqueness**: Students cannot be duplicated across the system
- **Multi-teacher Assignment**: Students can be assigned to multiple teachers without duplication
- **Class Transfers**: Clear rules for moving students between classes
- **Data Integrity**: Maintain relationships across teacher-student assignments

### Content Management
- **File Support**: PDF, Word, Excel, Images, Videos, and other educational materials
- **Visibility Control**: Teachers control which classes can access specific content
- **Version Control**: Support for content updates and revisions

### Security & Access Control
- **Role-based Access**: Strict permission boundaries for each user type
- **Environment Separation**: Production and testing environments must remain isolated
- **Data Protection**: Student and educational data must be securely managed

## Success Metrics (V1)

### Technical Metrics
- System uptime > 99%
- Page load times < 3 seconds
- API response times < 500ms
- Database query performance optimization

### User Adoption Metrics
- Teacher onboarding completion rate
- Student engagement with uploaded content
- Class creation and management efficiency
- Content upload success rates

### Business Metrics
- User satisfaction scores
- System reliability
- Feature usage analytics
- Support ticket resolution times

## Next Steps

1. **Development Setup**: Environment configuration for both .NET and Angular
2. **Database Design**: Entity modeling and relationship mapping
3. **Authentication System**: JWT-based security implementation
4. **Core API Development**: User management and content APIs
5. **Frontend Components**: Angular components for key user workflows
6. **Testing Strategy**: Unit, integration, and user acceptance testing
7. **Deployment Pipeline**: CI/CD setup for both environments

This system overview provides the foundation for detailed technical documentation and development planning for the eLearning Management System V1.
