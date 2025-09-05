# UIIC Website Integration Guide - IT Support Portal

## Overview
Integration guide for adding the IT Support Portal to UIIC's existing Drupal-based website.

## UIIC Current Technology Stack
- **CMS**: Drupal (PHP-based)
- **Backend**: PHP with Apache
- **Database**: MySQL/MariaDB
- **Frontend*: jQuery, jQuery Migrate
- **Development**: Managed by TCS
- **Architecture**: Traditional LAMP stack

## Integration Options for Drupal Environment

### Option 1: Drupal Custom Module (Recommended)
Create a custom Drupal module to integrate the IT support functionality natively.

#### Step 1: Create Drupal Module Structure
```
/modules/custom/it_support_portal/
├── it_support_portal.info.yml
├── it_support_portal.module
├── it_support_portal.routing.yml
├── src/
│   ├── Controller/
│   │   └── SupportPortalController.php
│   ├── Form/
│   │   ├── EmployeeTicketForm.php
│   │   └── VendorTicketForm.php
│   └── Entity/
│       └── SupportTicket.php
├── templates/
│   ├── employee-portal.html.twig
│   ├── vendor-portal.html.twig
│   └── admin-panel.html.twig
└── js/
    └── support-portal.js
```

#### Step 2: Module Info File (it_support_portal.info.yml)
```yaml
name: IT Support Portal
type: module
description: Employee and Vendor IT Support Ticket System
core_version_requirement: ^9 || ^10
package: Custom
dependencies:
  - drupal:node
  - drupal:user
  - drupal:field
```

#### Step 3: Controller Implementation
```php
<?php
// src/Controller/SupportPortalController.php
namespace Drupal\it_support_portal\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\Database\Connection;
use Symfony\Component\DependencyInjection\ContainerInterface;

class SupportPortalController extends ControllerBase {

  protected $database;

  public function __construct(Connection $database) {
    $this->database = $database;
  }

  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('database')
    );
  }

  public function employeePortal() {
    return [
      '#theme' => 'employee_portal',
      '#attached' => [
        'library' => ['it_support_portal/portal-styling']
      ],
    ];
  }

  public function vendorPortal() {
    return [
      '#theme' => 'vendor_portal',
      '#attached' => [
        'library' => ['it_support_portal/portal-styling']
      ],
    ];
  }

  public function adminPanel() {
    // Check admin permissions
    if (!$this->currentUser()->hasPermission('administer it support')) {
      throw new \Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException();
    }

    $tickets = $this->database->select('support_tickets', 't')
      ->fields('t')
      ->orderBy('created_at', 'DESC')
      ->execute()
      ->fetchAll();

    return [
      '#theme' => 'admin_panel',
      '#tickets' => $tickets,
      '#attached' => [
        'library' => ['it_support_portal/admin-styling']
      ],
    ];
  }
}
```

#### Step 4: Database Schema Installation
```php
<?php
// it_support_portal.install

function it_support_portal_schema() {
  $schema['support_tickets'] = [
    'description' => 'IT Support Tickets',
    'fields' => [
      'id' => [
        'type' => 'serial',
        'not null' => TRUE,
        'description' => 'Primary Key',
      ],
      'ticket_id' => [
        'type' => 'varchar',
        'length' => 20,
        'not null' => TRUE,
        'description' => 'Unique Ticket ID',
      ],
      'user_type' => [
        'type' => 'varchar',
        'length' => 20,
        'not null' => TRUE,
        'description' => 'Employee or Vendor',
      ],
      'emp_name' => [
        'type' => 'varchar',
        'length' => 255,
        'description' => 'Employee Name',
      ],
      'department' => [
        'type' => 'varchar',
        'length' => 255,
        'description' => 'Department',
      ],
      'vendor_name' => [
        'type' => 'varchar',
        'length' => 255,
        'description' => 'Vendor Name',
      ],
      'floor_no' => [
        'type' => 'varchar',
        'length' => 10,
        'not null' => TRUE,
        'description' => 'Floor Number',
      ],
      'mobile' => [
        'type' => 'varchar',
        'length' => 15,
        'not null' => TRUE,
        'description' => 'Mobile Number',
      ],
      'designation' => [
        'type' => 'varchar',
        'length' => 100,
        'not null' => TRUE,
        'description' => 'Designation',
      ],
      'severity' => [
        'type' => 'varchar',
        'length' => 20,
        'not null' => TRUE,
        'description' => 'Issue Severity',
      ],
      'issue_type' => [
        'type' => 'varchar',
        'length' => 20,
        'not null' => TRUE,
        'description' => 'Type of Issue',
      ],
      'system_subtype' => [
        'type' => 'varchar',
        'length' => 100,
        'description' => 'System Issue Subtype',
      ],
      'printer_subtype' => [
        'type' => 'varchar',
        'length' => 100,
        'description' => 'Printer Issue Subtype',
      ],
      'others_description' => [
        'type' => 'text',
        'description' => 'Other Issues Description',
      ],
      'status' => [
        'type' => 'varchar',
        'length' => 20,
        'not null' => TRUE,
        'default' => 'open',
        'description' => 'Ticket Status',
      ],
      'created_at' => [
        'type' => 'int',
        'not null' => TRUE,
        'description' => 'Creation timestamp',
      ],
    ],
    'primary key' => ['id'],
    'unique keys' => [
      'ticket_id' => ['ticket_id'],
    ],
  ];

  return $schema;
}
```

#### Step 5: Routing Configuration (it_support_portal.routing.yml)
```yaml
it_support_portal.employee:
  path: '/support/employee'
  defaults:
    _controller: '\Drupal\it_support_portal\Controller\SupportPortalController::employeePortal'
    _title: 'Employee IT Support'
  requirements:
    _permission: 'access content'

it_support_portal.vendor:
  path: '/support/vendor'
  defaults:
    _controller: '\Drupal\it_support_portal\Controller\SupportPortalController::vendorPortal'
    _title: 'Vendor IT Support'
  requirements:
    _permission: 'access content'

it_support_portal.admin:
  path: '/support/admin'
  defaults:
    _controller: '\Drupal\it_support_portal\Controller\SupportPortalController::adminPanel'
    _title: 'IT Support Admin Panel'
  requirements:
    _permission: 'administer it support'

# API endpoints for AJAX requests
it_support_portal.api.create_ticket:
  path: '/api/support/tickets'
  defaults:
    _controller: '\Drupal\it_support_portal\Controller\ApiController::createTicket'
  methods: [POST]
  requirements:
    _permission: 'access content'

it_support_portal.api.update_ticket:
  path: '/api/support/tickets/{ticket_id}/status'
  defaults:
    _controller: '\Drupal\it_support_portal\Controller\ApiController::updateTicketStatus'
  methods: [PATCH]
  requirements:
    _permission: 'administer it support'
```

#### Step 6: Twig Templates

**Employee Portal Template (templates/employee-portal.html.twig):**
```twig
<div class="support-portal employee-portal">
  <div class="portal-header">
    <h2>Employee IT Support Portal</h2>
    <p>Submit your IT support requests</p>
  </div>
  
  <form id="employee-support-form" class="support-form">
    <div class="form-row">
      <div class="form-group">
        <label for="emp_name">Employee Name *</label>
        <input type="text" id="emp_name" name="emp_name" required>
      </div>
    </div>
    
    <div class="form-row">
      <div class="form-group">
        <label for="floor_no">Floor Number *</label>
        <select id="floor_no" name="floor_no" required>
          <option value="">Select Floor</option>
          {% for i in 1..14 %}
            <option value="{{ i }}">{{ i }}{{ i == 1 ? 'st' : (i == 2 ? 'nd' : (i == 3 ? 'rd' : 'th')) }} Floor</option>
          {% endfor %}
        </select>
      </div>
      
      <div class="form-group">
        <label for="department">Department *</label>
        <input type="text" id="department" name="department" required>
      </div>
    </div>
    
    <div class="form-row">
      <div class="form-group">
        <label for="mobile">Mobile Number *</label>
        <input type="tel" id="mobile" name="mobile" placeholder="+91 9876543210" pattern="^\+91 [6-9]\d{9}$" required>
      </div>
      
      <div class="form-group">
        <label for="designation">Designation *</label>
        <select id="designation" name="designation" required>
          <option value="">Select Designation</option>
          <option value="assistant">Assistant</option>
          <option value="sr-assistant">Sr. Assistant</option>
          <option value="scale-1">Scale 1</option>
          <option value="scale-2">Scale 2</option>
          <option value="scale-3">Scale 3</option>
          <option value="scale-4">Scale 4</option>
          <option value="scale-5">Scale 5</option>
          <option value="scale-6">Scale 6</option>
          <option value="scale-7">Scale 7</option>
        </select>
      </div>
    </div>
    
    <div class="form-row">
      <div class="form-group">
        <label for="severity">Severity Level *</label>
        <select id="severity" name="severity" required>
          <option value="">Select Severity</option>
          <option value="low">Low - Minor inconvenience</option>
          <option value="medium">Medium - Affects productivity</option>
          <option value="high">High - Significant impact</option>
          <option value="critical">Critical - Work stoppage</option>
        </select>
      </div>
      
      <div class="form-group">
        <label for="issue_type">Issue Type *</label>
        <select id="issue_type" name="issue_type" required>
          <option value="">Select Issue Type</option>
          <option value="system">System Issues</option>
          <option value="printer">Printer Issues</option>
          <option value="others">Others</option>
        </select>
      </div>
    </div>
    
    <div id="system-subtypes" class="conditional-field" style="display: none;">
      <div class="form-group">
        <label for="system_subtype">System Issue Sub-type</label>
        <select id="system_subtype" name="system_subtype">
          <option value="">Select System Issue</option>
          <option value="system-slow">System Running Slow</option>
          <option value="network-issue">Network Connectivity</option>
          <option value="booting-issue">Booting Problems</option>
          <option value="blue-screen">Blue Screen Error</option>
          <option value="power">Power Issues</option>
          <option value="monitor">Monitor Problems</option>
          <option value="port">Port Not Working</option>
          <option value="keyboard">Keyboard Issues</option>
          <option value="mouse">Mouse Problems</option>
        </select>
      </div>
    </div>
    
    <div id="printer-subtypes" class="conditional-field" style="display: none;">
      <div class="form-group">
        <label for="printer_subtype">Printer Issue Sub-type</label>
        <select id="printer_subtype" name="printer_subtype">
          <option value="">Select Printer Issue</option>
          <option value="configuration">Configuration Problems</option>
          <option value="paper-jam">Paper Jam</option>
          <option value="usb-port">USB Port Issues</option>
          <option value="duplex-printing">Duplex Printing Problems</option>
        </select>
      </div>
    </div>
    
    <div id="others-description" class="conditional-field" style="display: none;">
      <div class="form-group">
        <label for="others_description">Issue Description</label>
        <textarea id="others_description" name="others_description" rows="4"></textarea>
      </div>
    </div>
    
    <div class="form-actions">
      <button type="submit" class="btn btn-primary">Submit Ticket</button>
    </div>
  </form>
</div>
```

#### Step 7: JavaScript Integration (js/support-portal.js)
```javascript
(function ($, Drupal) {
  'use strict';

  Drupal.behaviors.supportPortal = {
    attach: function (context, settings) {
      // Issue type conditional fields
      $('#issue_type', context).once('issue-type-handler').change(function() {
        var issueType = $(this).val();
        $('.conditional-field').hide();
        
        if (issueType === 'system') {
          $('#system-subtypes').show();
        } else if (issueType === 'printer') {
          $('#printer-subtypes').show();
        } else if (issueType === 'others') {
          $('#others-description').show();
        }
      });
      
      // Form submission
      $('#employee-support-form, #vendor-support-form', context).once('form-submit-handler').submit(function(e) {
        e.preventDefault();
        
        var formData = {
          user_type: $(this).attr('id').includes('employee') ? 'employee' : 'vendor'
        };
        
        // Collect form data
        $(this).find('input, select, textarea').each(function() {
          if ($(this).val()) {
            formData[$(this).attr('name')] = $(this).val();
          }
        });
        
        // Submit via AJAX
        $.ajax({
          url: '/api/support/tickets',
          method: 'POST',
          data: JSON.stringify(formData),
          contentType: 'application/json',
          headers: {
            'X-Requested-With': 'XMLHttpRequest',
            'X-CSRF-Token': $('meta[name="csrf-token"]').attr('content')
          },
          success: function(response) {
            showSuccessMessage('Ticket ' + response.ticket_id + ' created successfully!');
            $(this)[0].reset();
          }.bind(this),
          error: function(xhr) {
            showErrorMessage('Failed to create ticket. Please try again.');
          }
        });
      });
    }
  };

  function showSuccessMessage(message) {
    var $message = $('<div class="alert alert-success">' + message + '</div>');
    $('.support-portal').prepend($message);
    setTimeout(function() {
      $message.fadeOut();
    }, 5000);
  }

  function showErrorMessage(message) {
    var $message = $('<div class="alert alert-error">' + message + '</div>');
    $('.support-portal').prepend($message);
    setTimeout(function() {
      $message.fadeOut();
    }, 5000);
  }

})(jQuery, Drupal);
```

### Option 2: Drupal Block Integration
Create custom blocks that can be placed anywhere on the site.

#### Step 1: Block Plugin
```php
<?php
// src/Plugin/Block/SupportPortalBlock.php
namespace Drupal\it_support_portal\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Form\FormStateInterface;

/**
 * Provides an IT Support Portal block.
 *
 * @Block(
 *   id = "support_portal_block",
 *   admin_label = @Translation("IT Support Portal"),
 *   category = @Translation("Custom"),
 * )
 */
class SupportPortalBlock extends BlockBase {

  public function build() {
    return [
      '#theme' => 'support_portal_block',
      '#portal_type' => $this->configuration['portal_type'] ?? 'employee',
      '#attached' => [
        'library' => ['it_support_portal/portal-styling']
      ],
    ];
  }

  public function blockForm($form, FormStateInterface $form_state) {
    $form = parent::blockForm($form, $form_state);

    $form['portal_type'] = [
      '#type' => 'select',
      '#title' => $this->t('Portal Type'),
      '#options' => [
        'employee' => $this->t('Employee Portal'),
        'vendor' => $this->t('Vendor Portal'),
        'both' => $this->t('Both (Tabbed Interface)'),
      ],
      '#default_value' => $this->configuration['portal_type'] ?? 'employee',
    ];

    return $form;
  }

  public function blockSubmit($form, FormStateInterface $form_state) {
    $this->configuration['portal_type'] = $form_state->getValue('portal_type');
  }
}
```

### Option 3: Integration with Existing UIIC Navigation
Add the support portal as menu items in the existing UIIC navigation structure.

#### Step 1: Menu Links (it_support_portal.links.menu.yml)
```yaml
it_support_portal.employee:
  title: 'Employee IT Support'
  route_name: it_support_portal.employee
  description: 'Submit IT support requests for employees'
  parent: main
  weight: 100

it_support_portal.vendor:
  title: 'Vendor IT Support'
  route_name: it_support_portal.vendor
  description: 'Submit IT support requests for vendors'
  parent: main
  weight: 101
```

## CSS Styling Integration with UIIC Theme

```css
/* CSS that matches UIIC's existing design patterns */
.support-portal {
  background: #ffffff;
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: 20px;
  margin: 20px 0;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.portal-header {
  background: #1f4e79; /* UIIC blue theme */
  color: white;
  padding: 15px;
  margin: -20px -20px 20px -20px;
  border-radius: 4px 4px 0 0;
}

.support-form .form-row {
  display: flex;
  gap: 15px;
  margin-bottom: 15px;
}

.support-form .form-group {
  flex: 1;
}

.support-form label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
  color: #333;
}

.support-form input,
.support-form select,
.support-form textarea {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.btn-primary {
  background-color: #1f4e79;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.btn-primary:hover {
  background-color: #164060;
}

.alert {
  padding: 12px;
  margin-bottom: 15px;
  border-radius: 4px;
}

.alert-success {
  background-color: #d4edda;
  border-color: #c3e6cb;
  color: #155724;
}

.alert-error {
  background-color: #f8d7da;
  border-color: #f5c6cb;
  color: #721c24;
}
```

## Implementation Steps for UIIC

### Phase 1: Development Environment Setup
1. **Coordinate with TCS**: Since TCS manages UIIC's development, coordination is essential
2. **Create Development Branch**: Set up a separate branch for testing
3. **Database Backup**: Ensure proper backup before any schema changes

### Phase 2: Module Development
1. **Create Custom Module**: Following the structure provided above
2. **Database Schema**: Install the support_tickets table
3. **Permissions Setup**: Create appropriate user roles and permissions

### Phase 3: Testing
1. **Local Testing**: Test all functionality in development environment
2. **User Acceptance Testing**: Get approval from UIIC stakeholders
3. **Security Review**: Ensure compliance with UIIC security standards

### Phase 4: Deployment
1. **Staging Deployment**: Deploy to staging environment first
2. **Production Deployment**: Coordinate with TCS for production release
3. **User Training**: Train UIIC staff on the new system

## Security Considerations for UIIC

1. **User Authentication**: Integrate with UIIC's existing user system
2. **Data Encryption**: Ensure sensitive data is encrypted
3. **Access Control**: Role-based access for different user types
4. **Audit Logging**: Log all ticket creation and status changes
5. **Input Validation**: Server-side validation for all form inputs

## API Endpoints for Mobile Integration

```php
<?php
// For future mobile app integration
class ApiController extends ControllerBase {
  
  /**
   * GET /api/support/tickets
   * Returns tickets based on user permissions
   */
  public function getTickets(Request $request) {
    // Implementation for retrieving tickets
  }
  
  /**
   * POST /api/support/tickets
   * Creates a new support ticket
   */
  public function createTicket(Request $request) {
    // Implementation for creating tickets
  }
  
  /**
   * PATCH /api/support/tickets/{id}/status
   * Updates ticket status (admin only)
   */
  public function updateTicketStatus($ticket_id, Request $request) {
    // Implementation for status updates
  }
}
```

## Next Steps

1. **Contact TCS**: Reach out to UIIC's development team at TCS
2. **Requirements Gathering**: Discuss specific UIIC requirements
3. **Development Timeline**: Create a phased implementation plan
4. **Testing Strategy**: Plan comprehensive testing approach
5. **Training Plan**: Develop user training materials

This integration approach ensures the IT Support Portal fits seamlessly into UIIC's existing Drupal-based infrastructure while maintaining their security and design standards.