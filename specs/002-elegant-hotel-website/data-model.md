# Data Model: Elegant Oahu Hotel Website

**Feature**: Elegant Oahu Hotel Website  
**Date**: September 18, 2025  
**Context**: TypeScript interfaces for hotel booking domain with user authentication

## Core Entities

### User Entity
Represents registered account holders with authentication and personalization features.

```typescript
interface User {
  id: string;                    // UUID for unique identification
  email: string;                 // Primary authentication identifier
  passwordHash: string;          // Secure password storage (never plain text)
  profile: UserProfile;          // Personal information and preferences
  bookingHistory: BookingReference[]; // Array of past booking references
  preferences: UserPreferences;  // Saved user preferences
  createdAt: Date;              // Account creation timestamp
  lastLoginAt: Date;            // Last successful login timestamp
  isEmailVerified: boolean;     // Email verification status
  isActive: boolean;            // Account active status
}

interface UserProfile {
  firstName: string;            // Required for bookings
  lastName: string;             // Required for bookings
  phone?: string;               // Optional contact information
  dateOfBirth?: Date;           // Optional for age-based packages
  address?: Address;            // Optional billing/contact address
  emergencyContact?: EmergencyContact; // Optional for safety
}

interface UserPreferences {
  roomType?: RoomType;          // Preferred room type
  packageCategories?: PackageCategory[]; // Preferred package types
  dietaryRestrictions?: string[]; // Dietary needs for dining packages
  accessibilityNeeds?: string[]; // Accessibility requirements
  communicationPreferences: {   // How user wants to be contacted
    email: boolean;
    sms: boolean;
    marketing: boolean;
  };
}

interface Address {
  street: string;
  city: string;
  state: string;
  zipCode: string;
  country: string;
}

interface EmergencyContact {
  name: string;
  relationship: string;
  phone: string;
  email?: string;
}
```

**Validation Rules**:
- Email must be valid format and unique across all users
- Password must meet security requirements (8+ chars, mixed case, numbers, symbols)
- Phone numbers must be valid international format if provided
- User must be 18+ for booking creation (derived from dateOfBirth)

**State Transitions**:
- Registration → Email Verification → Active Account
- Active → Inactive (account suspension)
- Password Reset Request → Password Reset Complete → Active

### Guest Entity  
Represents non-registered visitors making bookings without account creation.

```typescript
interface Guest {
  id: string;                   // Temporary UUID for booking session
  contactInfo: ContactInfo;     // Required for booking confirmation
  specialRequests?: string[];   // Optional requests for stay
  emergencyContact?: EmergencyContact; // Optional emergency contact
  sessionId: string;            // Browser session identifier
  createdAt: Date;              // Guest record creation
}

interface ContactInfo {
  firstName: string;            // Required for reservation
  lastName: string;             // Required for reservation  
  email: string;                // Required for confirmation
  phone: string;                // Required for hotel contact
}
```

**Validation Rules**:
- All ContactInfo fields are required for booking completion
- Email must be valid format (not necessarily unique for guests)
- Phone must be valid format for hotel communication

### Room Entity
Represents hotel accommodations with availability, pricing, and feature details.

```typescript
interface Room {
  id: string;                   // Unique room identifier
  number: string;               // Hotel room number (e.g., "301", "Ocean Suite A")
  type: RoomType;               // Room category classification
  name: string;                 // Display name (e.g., "Deluxe Ocean View")
  description: string;          // Detailed room description
  capacity: RoomCapacity;       // Occupancy limits and bed configuration
  amenities: RoomAmenity[];     // Room-specific amenities
  images: RoomImage[];          // Photography for display
  pricing: RoomPricing;         // Rate structure and availability
  location: RoomLocation;       // Hotel floor, view, accessibility
  isActive: boolean;            // Available for booking
  maintenanceSchedule?: MaintenanceWindow[]; // Planned unavailability
}

type RoomType = 'standard' | 'deluxe' | 'suite' | 'penthouse' | 'family';

interface RoomCapacity {
  maxGuests: number;            // Maximum occupancy
  maxAdults: number;            // Adult guest limit
  maxChildren: number;          // Child guest limit  
  beds: BedConfiguration[];     // Bed types and quantities
  hasRollawayOption: boolean;   // Additional bed availability
}

interface BedConfiguration {
  type: 'king' | 'queen' | 'full' | 'twin' | 'sofa bed';
  quantity: number;
}

interface RoomAmenity {
  id: string;
  name: string;                 // Display name
  description?: string;         // Additional details
  category: AmenityCategory;    // Grouping for display
  isIncluded: boolean;          // Included vs. paid amenity
  additionalCost?: number;      // Cost if paid amenity
}

interface RoomImage {
  id: string;
  url: string;                  // Image URL
  alt: string;                  // Accessibility description
  caption?: string;             // Optional display caption
  isPrimary: boolean;           // Primary image for listings
  order: number;                // Display order
}

interface RoomPricing {
  baseRate: number;             // Nightly rate in USD
  currency: string;             // ISO currency code
  seasonalMultipliers: SeasonalRate[]; // Dynamic pricing
  minimumStay?: number;         // Minimum nights required
  advanceBookingDiscount?: number; // Early booking discount
}

interface SeasonalRate {
  startDate: Date;              // Season start
  endDate: Date;                // Season end
  multiplier: number;           // Rate adjustment (1.0 = base rate)
  name: string;                 // Season name (e.g., "Holiday Season")
}

interface RoomLocation {
  floor: number;                // Hotel floor number
  view: ViewType;               // Room view category
  section: string;              // Hotel section (e.g., "North Wing")
  isAccessible: boolean;        // ADA accessibility compliance
}

type ViewType = 'ocean' | 'garden' | 'courtyard' | 'city' | 'mountain';
```

**Validation Rules**:
- Room capacity cannot exceed safety regulations (maxGuests ≤ 4 for standard rooms)
- Pricing baseRate must be positive number
- At least one primary image required for active rooms
- Seasonal multipliers must cover full calendar year

### Booking Entity
Represents confirmed reservations linking users/guests to rooms with comprehensive details.

```typescript
interface Booking {
  id: string;                   // Unique booking reference
  confirmationCode: string;     // Guest-facing confirmation number
  status: BookingStatus;        // Current booking state
  guest: BookingGuest;          // Guest or user information
  room: BookingRoom;            // Reserved room details
  dates: BookingDates;          // Check-in/out and booking dates
  packages: BookingPackage[];   // Selected add-on packages
  pricing: BookingPricing;      // Complete cost breakdown
  paymentInfo: PaymentInfo;     // Payment status and details
  specialRequests?: string[];   // Guest requests and notes
  cancellationPolicy: CancellationPolicy; // Terms and deadlines
  modificationHistory: BookingModification[]; // Change log
  createdAt: Date;              // Initial booking timestamp
  updatedAt: Date;              // Last modification timestamp
}

type BookingStatus = 
  | 'pending'           // Payment pending
  | 'confirmed'         // Payment complete, reservation confirmed
  | 'checked-in'        // Guest has arrived
  | 'checked-out'       // Stay completed
  | 'cancelled'         // Booking cancelled
  | 'no-show';          // Guest did not arrive

interface BookingGuest {
  type: 'user' | 'guest';       // Account type
  userId?: string;              // Reference if registered user
  guestId?: string;             // Reference if anonymous guest
  contactInfo: ContactInfo;     // Contact details for reservation
  partySize: PartyDetails;      // Guest party composition
}

interface PartyDetails {
  adults: number;               // Adult guest count
  children: number;             // Child guest count
  infants: number;              // Infant count (may not count toward capacity)
  childAges?: number[];         // Child ages for appropriate accommodations
}

interface BookingRoom {
  roomId: string;               // Reference to Room entity
  roomNumber: string;           // Hotel room number
  roomType: RoomType;           // Room category
  roomName: string;             // Display name
  selectedAmenities: string[];  // Additional paid amenities
}

interface BookingDates {
  checkIn: Date;                // Arrival date
  checkOut: Date;               // Departure date  
  nights: number;               // Calculated stay duration
  bookingDate: Date;            // When reservation was made
  paymentDueDate: Date;         // Payment deadline
}

interface BookingPricing {
  roomTotal: number;            // Room rate × nights
  packageTotal: number;         // Sum of selected packages
  amenityTotal: number;         // Additional amenity costs
  subtotal: number;             // Before taxes and fees
  taxes: TaxBreakdown[];        // Tax details
  fees: FeeBreakdown[];         // Hotel fees
  discounts: DiscountBreakdown[]; // Applied discounts
  totalAmount: number;          // Final amount due
  currency: string;             // ISO currency code
}

interface TaxBreakdown {
  name: string;                 // Tax name (e.g., "State Tax")
  rate: number;                 // Tax percentage
  amount: number;               // Calculated tax amount
}

interface FeeBreakdown {
  name: string;                 // Fee name (e.g., "Resort Fee")
  amount: number;               // Fee amount
  isPerNight: boolean;          // Per night vs. one-time
}

interface DiscountBreakdown {
  name: string;                 // Discount name
  type: 'percentage' | 'fixed'; // Discount type
  value: number;                // Discount value
  amount: number;               // Applied discount amount
}

interface PaymentInfo {
  status: PaymentStatus;        // Payment state
  method?: PaymentMethod;       // How payment was made
  transactionId?: string;       // Payment processor reference
  paidAmount: number;           // Amount paid
  paidAt?: Date;                // Payment completion timestamp
  refundAmount?: number;        // Refunded amount if cancelled
  refundedAt?: Date;            // Refund completion timestamp
}

type PaymentStatus = 'pending' | 'completed' | 'failed' | 'refunded' | 'partial';
type PaymentMethod = 'credit_card' | 'debit_card' | 'bank_transfer' | 'cash';

interface CancellationPolicy {
  allowedUntil: Date;           // Latest cancellation date
  penaltyAmount: number;        // Cancellation fee
  refundPercentage: number;     // Percentage refunded if cancelled
  terms: string;                // Full cancellation terms text
}

interface BookingModification {
  id: string;                   // Modification record ID
  timestamp: Date;              // When change was made
  modifiedBy: string;           // User/guest who made change
  changes: ModificationDetail[]; // What was changed
  previousValues: Record<string, any>; // Original values
  newValues: Record<string, any>;      // Updated values
}

interface ModificationDetail {
  field: string;                // Field that changed
  oldValue: any;                // Previous value
  newValue: any;                // New value
  reason?: string;              // Reason for change
}
```

**Validation Rules**:
- Check-in date must be in the future
- Check-out date must be after check-in date
- Party size cannot exceed room capacity
- Payment amount must match calculated total
- Cancellation must respect policy deadlines

**State Transitions**:
- Pending → Confirmed (payment complete)
- Confirmed → Checked-in (guest arrival)
- Checked-in → Checked-out (guest departure)
- Any status → Cancelled (with policy validation)

### Package Entity
Represents add-on experiences and services that enhance guest stays.

```typescript
interface Package {
  id: string;                   // Unique package identifier
  name: string;                 // Package display name
  description: string;          // Detailed package description
  category: PackageCategory;    // Package type classification
  pricing: PackagePricing;      // Cost structure
  inclusions: PackageInclusion[]; // What's included
  restrictions: PackageRestriction[]; // Limitations and requirements
  availability: PackageAvailability; // When package is offered
  images: PackageImage[];       // Marketing imagery
  isActive: boolean;            // Available for purchase
  popularity: number;           // Sort order (1-10 scale)
  duration?: PackageDuration;   // Time commitment if applicable
}

type PackageCategory = 
  | 'romance'       // Couples experiences
  | 'family'        // Family-friendly activities
  | 'business'      // Business traveler amenities  
  | 'wellness'      // Spa and health packages
  | 'dining'        // Food and beverage experiences
  | 'adventure'     // Outdoor and activity packages
  | 'seasonal';     // Limited-time seasonal offerings

interface PackagePricing {
  basePrice: number;            // Package cost in USD
  currency: string;             // ISO currency code
  isPerPerson: boolean;         // Per person vs. flat rate
  isPerNight: boolean;          // Per night vs. one-time
  minimumPartySize?: number;    // Minimum guests required
  maximumPartySize?: number;    // Maximum guests allowed
  childDiscount?: number;       // Child pricing discount percentage
  advanceBookingDiscount?: number; // Early booking discount
}

interface PackageInclusion {
  name: string;                 // Included service/item name
  description: string;          // Details of inclusion
  value?: number;               // Retail value if applicable
  isHighlight: boolean;         // Feature prominently
}

interface PackageRestriction {
  type: RestrictionType;        // Type of limitation
  description: string;          // Human-readable restriction
  value?: string | number;      // Specific restriction value
}

type RestrictionType = 
  | 'age_minimum'       // Minimum age requirement
  | 'age_maximum'       // Maximum age requirement  
  | 'room_type'         // Compatible room types only
  | 'advance_booking'   // Must book X days ahead
  | 'season'            // Available certain seasons only
  | 'party_size'        // Party size limitations
  | 'health';           // Health/fitness requirements

interface PackageAvailability {
  isYearRound: boolean;         // Available all year
  seasonalPeriods?: SeasonalPeriod[]; // Limited availability
  blackoutDates?: DateRange[];  // Unavailable dates
  advanceBookingRequired: number; // Days ahead booking required
  cancellationDeadline: number; // Days before check-in
}

interface SeasonalPeriod {
  startDate: Date;              // Availability start
  endDate: Date;                // Availability end
  name: string;                 // Period name
}

interface DateRange {
  startDate: Date;
  endDate: Date;
  reason: string;               // Why unavailable
}

interface PackageImage {
  id: string;
  url: string;                  // Image URL
  alt: string;                  // Accessibility description
  caption?: string;             // Display caption
  isPrimary: boolean;           // Primary package image
  order: number;                // Display order
}

interface PackageDuration {
  hours?: number;               // Duration in hours
  days?: number;                // Duration in days
  isFlexible: boolean;          // Flexible timing
  recommendedTimeOfDay?: 'morning' | 'afternoon' | 'evening' | 'any';
}
```

**Validation Rules**:
- Package pricing must be positive
- If per-person pricing, party size limits must be specified
- Seasonal periods cannot overlap
- Advance booking required cannot exceed 365 days

### Amenity Entity
Represents hotel facilities and services available to guests.

```typescript
interface Amenity {
  id: string;                   // Unique amenity identifier
  name: string;                 // Amenity display name
  description: string;          // Detailed description
  category: AmenityCategory;    // Amenity classification
  location: AmenityLocation;    // Where amenity is located
  availability: AmenityAvailability; // Hours and seasonal availability
  pricing: AmenityPricing;      // Cost structure if applicable
  images: AmenityImage[];       // Visual representation
  features: AmenityFeature[];   // Specific amenity features
  accessibilityInfo: AccessibilityInfo; // Accessibility details
  isActive: boolean;            // Currently available
  popularityRating: number;     // Guest rating (1-5 scale)
}

type AmenityCategory = 
  | 'recreation'        // Pools, fitness, activities
  | 'dining'           // Restaurants, bars, room service
  | 'business'         // Meeting rooms, business center
  | 'wellness'         // Spa, salon, health services
  | 'convenience'      // Concierge, laundry, shopping
  | 'entertainment'    // Live music, shows, nightlife
  | 'family';          // Kids club, playground, family activities

interface AmenityLocation {
  building: string;             // Building name/number
  floor?: number;               // Floor number if applicable
  section: string;              // Area description (e.g., "Poolside", "Lobby Level")
  coordinates?: Coordinates;    // GPS coordinates if outdoor
  directions: string;           // How to find from main lobby
}

interface Coordinates {
  latitude: number;
  longitude: number;
}

interface AmenityAvailability {
  schedule: OperatingSchedule[];  // Daily hours
  seasonalClosures?: DateRange[]; // Temporary closures
  reservationRequired: boolean;   // Must book in advance
  capacityLimit?: number;         // Maximum guests
  ageRestrictions?: AgeRestriction; // Age requirements
}

interface OperatingSchedule {
  dayOfWeek: number;            // 0 = Sunday, 6 = Saturday
  openTime: string;             // Opening time (24hr format)
  closeTime: string;            // Closing time (24hr format)
  isClosed: boolean;            // Closed this day
  notes?: string;               // Special notes (e.g., "Last entry 1 hour before close")
}

interface AgeRestriction {
  minimumAge?: number;          // Minimum age for use
  maximumAge?: number;          // Maximum age if applicable
  requiresAdultSupervision: boolean; // Child supervision required
  adultSupervisionAge: number;  // Age considered adult for supervision
}

interface AmenityPricing {
  isFree: boolean;              // No charge for guests
  guestRate?: number;           // Rate for hotel guests
  nonGuestRate?: number;        // Rate for non-guests (if applicable)
  currency: string;             // ISO currency code
  billingType: 'per_use' | 'per_hour' | 'per_day' | 'per_person';
  deposit?: number;             // Security deposit if required
}

interface AmenityImage {
  id: string;
  url: string;                  // Image URL
  alt: string;                  // Accessibility description
  caption?: string;             // Display caption
  isPrimary: boolean;           // Primary amenity image
  order: number;                // Display order
  showInGallery: boolean;       // Include in photo gallery
}

interface AmenityFeature {
  name: string;                 // Feature name
  description: string;          // Feature description
  isHighlight: boolean;         // Emphasize in listings
  additionalCost?: number;      // Extra cost if premium feature
}

interface AccessibilityInfo {
  isWheelchairAccessible: boolean; // ADA compliant access
  hasAssistiveListening: boolean;  // Hearing assistance available
  hasVisualAids: boolean;          // Visual assistance available
  accessibilityNotes: string;     // Additional accessibility information
}
```

**Validation Rules**:
- Operating schedule must not have overlapping times
- Age restrictions must be logical (minimum < maximum)
- Pricing must be positive if not free
- At least one primary image required for active amenities

### Property Entity
Represents the Oahu hotel location with comprehensive information.

```typescript
interface Property {
  id: string;                   // Property identifier
  name: string;                 // Official hotel name
  brandName?: string;           // Hotel brand if applicable
  description: string;          // Property description
  location: PropertyLocation;   // Geographic and address information
  contact: PropertyContact;     // Communication details
  policies: PropertyPolicies;   // Hotel policies and rules
  certifications: Certification[]; // Awards and certifications
  culturalInfo: CulturalInfo;   // Hawaiian cultural context
  images: PropertyImage[];      // Property photography
  socialMedia: SocialMediaLinks; // Online presence
  isActive: boolean;            // Property operational status
}

interface PropertyLocation {
  address: Address;             // Physical address
  coordinates: Coordinates;     // GPS location
  timezone: string;             // IANA timezone identifier
  nearbyLandmarks: Landmark[];  // Points of interest
  transportation: TransportationInfo; // Getting to property
  weatherInfo: WeatherInfo;     // Climate information
}

interface Landmark {
  name: string;                 // Landmark name
  type: LandmarkType;           // Category
  distance: number;             // Distance in miles
  estimatedTravelTime: string;  // Travel time estimate
  description: string;          // Additional details
}

type LandmarkType = 
  | 'beach'         // Beaches and coastal areas
  | 'attraction'    // Tourist attractions
  | 'shopping'      // Shopping centers and districts
  | 'dining'        // Notable restaurants and food areas
  | 'transportation' // Airports, harbors, transit
  | 'cultural'      // Cultural sites and museums
  | 'nature';       // Parks, hiking trails, natural areas

interface TransportationInfo {
  airportDistance: number;      // Miles to primary airport
  airportName: string;          // Airport name
  shuttleService: boolean;      // Hotel provides shuttle
  publicTransit: PublicTransitInfo; // Public transportation options
  parkingInfo: ParkingInfo;     // Parking availability and cost
  walkabilityScore: number;     // Walkability rating (1-10)
}

interface PublicTransitInfo {
  isAvailable: boolean;         // Public transit accessible
  nearestStop?: string;         // Closest transit stop
  transitTypes: string[];       // Types available (bus, rail, etc.)
}

interface ParkingInfo {
  isAvailable: boolean;         // Parking available
  capacity?: number;            // Parking spaces
  cost: number;                 // Daily parking rate
  isValet: boolean;             // Valet parking service
  isSecured: boolean;           // Secure parking facility
}

interface WeatherInfo {
  averageTemperature: {         // Typical temperatures
    high: number;               // Average high (Fahrenheit)
    low: number;                // Average low (Fahrenheit)
  };
  rainySeasonMonths: number[];  // Months with higher rainfall
  bestVisitMonths: number[];    // Recommended visit months
  temperatureUnit: 'F' | 'C';   // Temperature measurement unit
}

interface PropertyContact {
  phone: string;                // Main phone number
  fax?: string;                 // Fax number if available
  email: string;                // General inquiry email
  website: string;              // Official website URL
  reservationsPhone: string;    // Booking phone number
  reservationsEmail: string;    // Booking email
  emergencyPhone: string;       // 24/7 emergency contact
}

interface PropertyPolicies {
  checkInTime: string;          // Standard check-in time
  checkOutTime: string;         // Standard check-out time
  cancellationPolicy: string;   // Cancellation terms
  petPolicy: PetPolicy;         // Pet accommodation rules
  smokingPolicy: string;        // Smoking restrictions
  minimumAge: number;           // Minimum age for booking
  maximumOccupancy: number;     // Property guest limit
  quietHours: string;           // Noise restriction hours
}

interface PetPolicy {
  petsAllowed: boolean;         // Pets permitted
  petFee?: number;              // Pet accommodation fee
  petDeposit?: number;          // Pet security deposit
  petRestrictions: string;      // Size, type, behavior restrictions
  petAmenities: string[];       // Pet-friendly amenities
}

interface Certification {
  name: string;                 // Certification/award name
  issuingOrganization: string;  // Who granted certification
  dateAwarded: Date;            // When received
  expirationDate?: Date;        // If certification expires
  description: string;          // What certification represents
  logoUrl?: string;             // Certification logo
}

interface CulturalInfo {
  culturalSignificance: string;  // Hawaiian cultural context
  respectfulPractices: string[]; // Cultural guidelines for guests
  localTraditions: string;       // Oahu traditions and customs
  culturalActivities: string[];  // Cultural experiences offered
  communityPartnerships: string[]; // Local community relationships
}

interface PropertyImage {
  id: string;
  url: string;                  // Image URL
  alt: string;                  // Accessibility description
  caption?: string;             // Display caption
  category: PropertyImageCategory; // Image classification
  isPrimary: boolean;           // Primary property image
  order: number;                // Display order
  showOnHomepage: boolean;      // Feature on homepage
}

type PropertyImageCategory = 
  | 'exterior'      // Building exterior and grounds
  | 'lobby'         // Lobby and common areas
  | 'amenities'     // Property amenities
  | 'views'         // Property views and scenery
  | 'cultural'      // Hawaiian cultural elements
  | 'events';       // Special events and activities

interface SocialMediaLinks {
  facebook?: string;            // Facebook page URL
  instagram?: string;           // Instagram account URL
  twitter?: string;             // Twitter account URL
  youtube?: string;             // YouTube channel URL
  tripadvisor?: string;         // TripAdvisor listing URL
}
```

**Validation Rules**:
- Contact information must include valid phone numbers and email addresses
- Coordinates must be valid latitude/longitude for Oahu
- Check-in time must be before check-out time
- All URLs must be valid and accessible

### Session Entity
Represents user authentication sessions and security state.

```typescript
interface Session {
  id: string;                   // Session identifier
  userId?: string;              // User ID if authenticated
  guestId?: string;             // Guest ID if anonymous
  sessionToken: string;         // Secure session token
  refreshToken?: string;        // Token refresh capability
  isAuthenticated: boolean;     // Authentication status
  loginTimestamp?: Date;        // When user logged in
  lastActivityTimestamp: Date;  // Last session activity
  expiresAt: Date;              // Session expiration time
  ipAddress: string;            // Client IP address
  userAgent: string;            // Browser/device information
  deviceInfo: DeviceInfo;       // Device details
  securityFlags: SecurityFlags; // Security status indicators
  bookingDraft?: BookingDraft;  // In-progress booking
}

interface DeviceInfo {
  deviceType: 'desktop' | 'mobile' | 'tablet'; // Device category
  operatingSystem: string;      // OS name and version
  browser: string;              // Browser name and version
  screenResolution?: string;    // Screen resolution if available
  timezone: string;             // Client timezone
  language: string;             // Preferred language
}

interface SecurityFlags {
  isSuspicious: boolean;        // Flagged for unusual activity
  requiresPasswordChange: boolean; // Password reset required
  multipleLoginAttempts: number; // Failed login count
  isFromKnownDevice: boolean;   // Device previously used
  hasCompletedTwoFactor: boolean; // 2FA completed if enabled
}

interface BookingDraft {
  selectedRoomId?: string;      // Room selection in progress
  selectedDates?: {             // Date selection
    checkIn: Date;
    checkOut: Date;
  };
  selectedPackages: string[];   // Package selections
  guestInfo?: Partial<ContactInfo>; // Partially completed guest info
  specialRequests?: string[];   // Guest requests
  lastUpdated: Date;            // When draft was modified
  expiresAt: Date;              // Draft expiration time
}
```

**Validation Rules**:
- Session tokens must be cryptographically secure
- Session expiration must not exceed security policy limits (typically 8 hours)
- Booking drafts expire after 30 minutes of inactivity
- IP address and user agent must be valid format

**State Transitions**:
- Anonymous → Authenticated (successful login)
- Authenticated → Expired (timeout reached)
- Active → Suspicious (security flags triggered)
- Any state → Terminated (explicit logout)

## Entity Relationships

```
User (1) ←→ (0..n) Booking
Guest (1) ←→ (0..n) Booking  
Room (1) ←→ (0..n) Booking
Booking (1) ←→ (0..n) BookingPackage ←→ (1) Package
Property (1) ←→ (0..n) Room
Property (1) ←→ (0..n) Amenity
User (1) ←→ (0..n) Session
Guest (1) ←→ (0..1) Session
```

## Data Validation Summary

All entities include comprehensive validation rules covering:
- Required field validation
- Format validation (email, phone, URLs)
- Business logic validation (dates, capacities, pricing)
- Security validation (authentication, authorization)
- Cross-entity referential integrity

## Mock Data Strategy

For initial implementation:
- Generate realistic Hawaiian hotel data
- Include diverse room types and packages representative of Oahu luxury hotels
- Create sample user accounts and booking scenarios
- Implement localStorage persistence for user sessions
- Provide data migration path for future database integration

**Status**: Phase 1 Data Modeling Complete - Ready for Service Contracts