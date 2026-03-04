# Implementation Guide - Admin Portal Enhancements

## File-by-File Changes

### 1. `/src/data/mockAppointments.ts`

#### Change: Added MongoDB `_id` field to Appointment interface

```typescript
// Before
export interface Appointment {
  id: string;
  fullName: string;
  // ...
}

// After
export interface Appointment {
  _id?: string;
  id?: string;
  fullName: string;
  // ...
}
```

#### Change: Added `_id` to all mock appointment records

```typescript
// Example - APT-001
{
  _id: "507f1f77bcf86cd799439011",  // New MongoDB ID
  id: "APT-001",
  fullName: "James Morrison",
  // ... rest of fields
}
```

**Why:** MongoDB stores records with `_id` field. Backend API uses this for DELETE/PUT operations.

---

### 2. `/src/pages/AdminDashboard.tsx`

#### Change 1: Updated Appointment interface to support both ID types

```typescript
// Safe ID extraction in multiple places
const searchId = a._id || a.id || '';
const aptId = apt._id || apt.id || '';
```

#### Change 2: Added delete confirmation state

```typescript
const [deleteConfirm, setDeleteConfirm] = useState<{ 
  type: 'appointment' | 'user' | 'coupon' | null; 
  id: string 
}>({ type: null, id: '' });
const [isDeleting, setIsDeleting] = useState(false);
```

#### Change 3: Added appointment update and delete handlers

```typescript
const handleUpdateAppointment = async (appointmentId: string) => {
  if (!editStatus) {
    toast.error('Please select a status');
    return;
  }

  try {
    const response = await fetch(
      `http://localhost:3000/api/appointments?id=${appointmentId}`,
      {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({ status: editStatus })
      }
    );
    const data = await response.json();
    if (data.success) {
      toast.success('Appointment updated successfully');
      setEditApt(null);
      setEditStatus('');
      fetchAppointments();
    } else {
      toast.error(data.message || 'Failed to update appointment');
    }
  } catch (error) {
    console.error("[v0] Error updating appointment:", error);
    toast.error('Error updating appointment');
  }
};

const handleDeleteAppointment = async (appointmentId: string) => {
  setDeleteConfirm({ type: 'appointment', id: appointmentId });
};
```

#### Change 4: Centralized delete confirmation handler

```typescript
const confirmDelete = async () => {
  if (!deleteConfirm.type || !deleteConfirm.id) return;
  
  setIsDeleting(true);
  try {
    let endpoint = '';
    switch (deleteConfirm.type) {
      case 'appointment':
        endpoint = `http://localhost:3000/api/appointments?id=${deleteConfirm.id}`;
        break;
      case 'user':
        endpoint = `http://localhost:3000/api/users?id=${deleteConfirm.id}`;
        break;
      case 'coupon':
        endpoint = `http://localhost:3000/api/coupons?id=${deleteConfirm.id}`;
        break;
    }

    const response = await fetch(endpoint, {
      method: 'DELETE',
      headers: { 'Authorization': `Bearer ${token}` }
    });
    
    const data = await response.json();
    if (data.success) {
      toast.success(`${deleteConfirm.type.charAt(0).toUpperCase() + deleteConfirm.type.slice(1)} deleted successfully`);
      setDeleteConfirm({ type: null, id: '' });
      
      if (deleteConfirm.type === 'appointment') fetchAppointments();
      else if (deleteConfirm.type === 'user') fetchUsers();
      else if (deleteConfirm.type === 'coupon') fetchCoupons();
    } else {
      toast.error(data.message || `Failed to delete ${deleteConfirm.type}`);
    }
  } catch (error) {
    console.error("[v0] Error deleting:", error);
    toast.error(`Error deleting ${deleteConfirm.type}`);
  } finally {
    setIsDeleting(false);
  }
};
```

#### Change 5: Added user status toggle handler

```typescript
const handleToggleUserStatus = async (userId: string, currentStatus: boolean) => {
  try {
    const response = await fetch(
      `http://localhost:3000/api/users?id=${userId}`,
      {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({ isActive: !currentStatus })
      }
    );
    const data = await response.json();
    if (data.success) {
      toast.success('User status updated');
      fetchUsers();
    } else {
      toast.error(data.message || 'Failed to update user');
    }
  } catch (error) {
    console.error("[v0] Error toggling user status:", error);
    toast.error('Error updating user');
  }
};
```

#### Change 6: Updated appointment table delete button

```typescript
// Before
<button onClick={() => handleDelete(apt.id)} ...>

// After
<button onClick={() => handleDeleteAppointment(apt._id || apt.id || '')} ...>
```

#### Change 7: Updated appointment edit dialog save button

```typescript
// Before
<Button onClick={handleEditSave} ...>Save Changes</Button>

// After
<Button onClick={() => handleUpdateAppointment(editApt._id || editApt.id || '')} ...>
  Save Changes
</Button>
```

#### Change 8: Added user status toggle switch in users table

```typescript
// Before
<button onClick={() => { setEditingUser(user); ... }} ...>
  <Pencil className="w-4 h-4" />
</button>

// After
<div className="flex items-center gap-2">
  <Switch 
    checked={user.isActive} 
    onCheckedChange={() => handleToggleUserStatus(user._id, user.isActive)} 
    className="data-[state=checked]:bg-emerald-500" 
  />
  <button onClick={() => { setEditingUser(user); ... }} ...>
    <Pencil className="w-4 h-4" />
  </button>
  // ... delete button
</div>
```

#### Change 9: Updated delete user handler (to use confirmation dialog)

```typescript
// Before
const handleDeleteUser = async (userId: string) => {
  if (!window.confirm('Are you sure...')) return;
  // ... fetch and delete
};

// After
const handleDeleteUser = async (userId: string) => {
  setDeleteConfirm({ type: 'user', id: userId });
};
```

#### Change 10: Updated delete coupon handler (same pattern)

```typescript
// Before: Uses window.confirm
// After: Uses setDeleteConfirm({ type: 'coupon', id: couponId })
```

#### Change 11: Added Delete Confirmation Dialog JSX

```typescript
{/* Delete Confirmation Dialog */}
<Dialog open={deleteConfirm.type !== null} onOpenChange={() => setDeleteConfirm({ type: null, id: '' })}>
  <DialogContent className="bg-card border-border text-foreground max-w-sm">
    <DialogHeader>
      <DialogTitle className="font-display text-xl text-red-400">Confirm Deletion</DialogTitle>
      <DialogDescription>This action cannot be undone. Are you sure?</DialogDescription>
    </DialogHeader>
    <div className="text-sm text-muted-foreground">
      You are about to permanently delete this {deleteConfirm.type}. This cannot be reversed.
    </div>
    <DialogFooter>
      <Button variant="outline" onClick={() => setDeleteConfirm({ type: null, id: '' })} disabled={isDeleting}>
        Cancel
      </Button>
      <Button onClick={confirmDelete} disabled={isDeleting} className="bg-red-500 text-white hover:bg-red-600">
        {isDeleting ? 'Deleting...' : 'Delete'}
      </Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

---

## API Integration Points

### Appointment Operations
```
PUT /api/appointments?id={_id}
Body: { status: "Pending|Confirmed|Completed|Cancelled" }
Response: { success: true, data: updatedAppointment }

DELETE /api/appointments?id={_id}
Response: { success: true, message: "Appointment deleted" }
```

### User Operations
```
PUT /api/users?id={_id}
Body: { fullName?: string, role?: string, isActive?: boolean }
Response: { success: true, user: updatedUser }

DELETE /api/users?id={_id}
Response: { success: true, message: "User deleted" }
```

### Coupon Operations
```
PUT /api/coupons?id={_id}
Body: { discountPercentage?: number, expiryDate?: string, isActive?: boolean }
Response: { success: true, coupon: updatedCoupon }

DELETE /api/coupons?id={_id}
Response: { success: true, message: "Coupon deleted" }
```

---

## State Management Changes

### New State Variables Added
```typescript
// Delete confirmation
const [deleteConfirm, setDeleteConfirm] = useState({ type: null, id: '' });
const [isDeleting, setIsDeleting] = useState(false);
```

### State Flow for Delete Operations
1. User clicks delete button
2. Handler sets `deleteConfirm` state, opening dialog
3. User confirms in dialog
4. `confirmDelete()` executes API call with `isDeleting: true`
5. After response, state resets to `{ type: null, id: '' }`

---

## Error Handling

### Pattern Used Throughout
```typescript
try {
  // API call with Authorization header
  const response = await fetch(endpoint, {
    method: 'PUT/DELETE',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    }
  });
  
  const data = await response.json();
  
  if (data.success) {
    // Show success toast
    toast.success('Action completed');
    // Refresh data
    fetchData();
  } else {
    // Show backend error
    toast.error(data.message || 'Action failed');
  }
} catch (error) {
  // Show client error
  console.error('Error:', error);
  toast.error('Error performing action');
}
```

---

## Performance Optimizations

1. **useMemo for calculations**
   ```typescript
   const filtered = useMemo(() => {
     if (!appointments || !Array.isArray(appointments)) return [];
     return appointments.filter(...);
   }, [appointments, statusFilter, search]);
   ```

2. **Safe optional chaining**
   ```typescript
   const id = apt._id || apt.id || '';
   ```

3. **Loading states to prevent duplicate submissions**
   ```typescript
   <Button disabled={isDeleting} onClick={confirmDelete}>
     {isDeleting ? 'Deleting...' : 'Delete'}
   </Button>
   ```

---

## Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires JavaScript enabled
- LocalStorage used for theme persistence
- No IE11 support

---

## Database Requirements

Appointments, Users, and Coupons must have MongoDB ObjectID (`_id`) field.
Example format: `507f1f77bcf86cd799439011`

---

## Security Considerations

1. All requests include `Authorization: Bearer {token}` header
2. Backend validates admin role before allowing CRUD operations
3. Users cannot delete their own accounts
4. Users cannot deactivate their own accounts
5. All inputs are validated on backend
6. Sensitive data (passwords) not displayed in frontend

