# Service Contracts: Elegant Oahu Hotel Website

**Feature**: Elegant Oahu Hotel Website  
**Date**: September 18, 2025  
**Context**: Mock service interfaces with TypeScript contracts for future database integration

## Service Architecture

The application follows a service-oriented architecture with clear contracts between the UI layer and data layer. All services use TypeScript interfaces for type safety and provide mock implementations for development.

## Authentication Service

### Interface Definition
```typescript
interface AuthenticationService {
  // User registration
  register(userData: UserRegistrationData): Promise<ServiceResponse<AuthResult>>;
  
  // User login
  login(credentials: LoginCredentials): Promise<ServiceResponse<AuthResult>>;
  
  // User logout
  logout(sessionId: string): Promise<ServiceResponse<void>>;
  
  // Password reset request
  requestPasswordReset(email: string): Promise<ServiceResponse<void>>;
  
  // Password reset completion
  resetPassword(resetData: PasswordResetData): Promise<ServiceResponse<void>>;
  
  // Session validation
  validateSession(sessionToken: string): Promise<ServiceResponse<SessionInfo>>;
  
  // Session refresh
  refreshSession(refreshToken: string): Promise<ServiceResponse<AuthResult>>;
}

interface UserRegistrationData {
  email: string;
  password: string;
  profile: {
    firstName: string;
    lastName: string;
    phone?: string;
  };
  preferences?: Partial<UserPreferences>;
}

interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

interface AuthResult {
  user: User;
  session: Session;
  accessToken: string;
  refreshToken?: string;
}

interface PasswordResetData {
  resetToken: string;
  newPassword: string;
}

interface SessionInfo {
  user: User;
  session: Session;
  isValid: boolean;
}
```

### Mock Implementation Strategy
```typescript
class MockAuthenticationService implements AuthenticationService {
  private users: Map<string, User> = new Map();
  private sessions: Map<string, Session> = new Map();
  private resetTokens: Map<string, string> = new Map(); // token -> email
  
  // Implementation generates realistic delays and handles edge cases
  // Uses localStorage to persist user data across browser sessions  
  // Implements proper password hashing simulation
  // Handles validation errors with detailed error messages
}
```

## Room Service

### Interface Definition
```typescript
interface RoomService {
  // Get all available rooms for date range
  getAvailableRooms(checkIn: Date, checkOut: Date): Promise<ServiceResponse<Room[]>>;
  
  // Get detailed room information
  getRoomDetails(roomId: string): Promise<ServiceResponse<Room>>;
  
  // Search rooms by criteria
  searchRooms(criteria: RoomSearchCriteria): Promise<ServiceResponse<Room[]>>;
  
  // Get room pricing for specific dates
  getRoomPricing(roomId: string, checkIn: Date, checkOut: Date): Promise<ServiceResponse<PricingResult>>;
  
  // Check room availability
  checkAvailability(roomId: string, checkIn: Date, checkOut: Date): Promise<ServiceResponse<AvailabilityResult>>;
}

interface RoomSearchCriteria {
  checkIn: Date;
  checkOut: Date;
  roomTypes?: RoomType[];
  maxGuests?: number;
  priceRange?: {
    min: number;
    max: number;
  };
  amenities?: string[];
  viewTypes?: ViewType[];
  accessibilityRequired?: boolean;
}

interface PricingResult {
  roomId: string;
  baseRate: number;
  totalRate: number;
  nights: number;
  seasonalMultiplier: number;
  discounts: DiscountBreakdown[];
  taxes: TaxBreakdown[];
  currency: string;
}

interface AvailabilityResult {
  roomId: string;
  isAvailable: boolean;
  availableDates: DateRange[];
  unavailableReason?: string;
  alternativeDates?: Date[];
}
```

### Mock Implementation Strategy
```typescript
class MockRoomService implements RoomService {
  private rooms: Room[] = []; // Pre-populated with Hawaiian hotel rooms
  private bookings: Booking[] = []; // Existing bookings for availability checking
  
  // Implementation includes realistic Oahu hotel room data
  // Simulates seasonal pricing variations
  // Handles availability conflicts with existing bookings
  // Returns appropriate errors for invalid date ranges
}
```

## Package Service

### Interface Definition
```typescript
interface PackageService {
  // Get all available packages
  getAllPackages(): Promise<ServiceResponse<Package[]>>;
  
  // Get packages by category
  getPackagesByCategory(category: PackageCategory): Promise<ServiceResponse<Package[]>>;
  
  // Get package details
  getPackageDetails(packageId: string): Promise<ServiceResponse<Package>>;
  
  // Check package compatibility with room and dates
  checkPackageCompatibility(
    packageId: string, 
    roomId: string, 
    dates: BookingDates,
    partySize: PartyDetails
  ): Promise<ServiceResponse<CompatibilityResult>>;
  
  // Get package pricing for specific scenario
  getPackagePricing(
    packageId: string,
    partySize: PartyDetails,
    nights: number
  ): Promise<ServiceResponse<PackagePricingResult>>;
}

interface CompatibilityResult {
  packageId: string;
  isCompatible: boolean;
  incompatibilityReasons?: string[];
  alternativePackages?: Package[];
  requiresUpgrade?: {
    roomType: RoomType;
    additionalCost: number;
  };
}

interface PackagePricingResult {
  packageId: string;
  basePrice: number;
  totalPrice: number;
  priceBreakdown: {
    adults: number;
    children: number;
    nights: number;
  };
  discounts: DiscountBreakdown[];
  currency: string;
}
```

### Mock Implementation Strategy
```typescript
class MockPackageService implements PackageService {
  private packages: Package[] = []; // Hawaiian-themed packages
  
  // Implementation includes authentic Hawaiian experiences
  // Romance packages for couples, family packages for children
  // Cultural packages highlighting Hawaiian traditions
  // Seasonal packages for holidays and events
}
```

## Booking Service

### Interface Definition
```typescript
interface BookingService {
  // Create new booking
  createBooking(bookingData: CreateBookingData): Promise<ServiceResponse<Booking>>;
  
  // Get booking details
  getBooking(bookingId: string): Promise<ServiceResponse<Booking>>;
  
  // Get bookings for user
  getUserBookings(userId: string): Promise<ServiceResponse<Booking[]>>;
  
  // Modify existing booking
  modifyBooking(bookingId: string, modifications: BookingModifications): Promise<ServiceResponse<Booking>>;
  
  // Cancel booking
  cancelBooking(bookingId: string, reason?: string): Promise<ServiceResponse<CancellationResult>>;
  
  // Calculate booking total
  calculateBookingTotal(bookingData: BookingCalculationData): Promise<ServiceResponse<BookingPricing>>;
  
  // Validate booking data
  validateBookingData(bookingData: CreateBookingData): Promise<ServiceResponse<ValidationResult>>;
}

interface CreateBookingData {
  guestInfo: BookingGuest;
  roomId: string;
  dates: {
    checkIn: Date;
    checkOut: Date;
  };
  packageIds: string[];
  specialRequests?: string[];
  paymentInfo?: PaymentInfo;
}

interface BookingModifications {
  dates?: {
    checkIn: Date;
    checkOut: Date;
  };
  packageIds?: string[];
  specialRequests?: string[];
  guestInfo?: Partial<BookingGuest>;
}

interface CancellationResult {
  bookingId: string;
  cancellationCode: string;
  refundAmount: number;
  refundMethod: PaymentMethod;
  refundTimeline: string;
  cancellationFee: number;
}

interface BookingCalculationData {
  roomId: string;
  checkIn: Date;
  checkOut: Date;
  packageIds: string[];
  partySize: PartyDetails;
  discountCodes?: string[];
}

interface ValidationResult {
  isValid: boolean;
  errors: ValidationError[];
  warnings: ValidationWarning[];
}

interface ValidationError {
  field: string;
  message: string;
  code: string;
}

interface ValidationWarning {
  field: string;
  message: string;
  severity: 'low' | 'medium' | 'high';
}
```

### Mock Implementation Strategy
```typescript
class MockBookingService implements BookingService {
  private bookings: Map<string, Booking> = new Map();
  
  // Implementation handles complex booking logic
  // Validates room availability before booking creation
  // Calculates accurate pricing including taxes and fees
  // Manages booking state transitions properly
  // Handles cancellation policies and refund calculations
}
```

## User Service

### Interface Definition
```typescript
interface UserService {
  // Get user profile
  getUserProfile(userId: string): Promise<ServiceResponse<User>>;
  
  // Update user profile
  updateUserProfile(userId: string, updates: Partial<UserProfile>): Promise<ServiceResponse<User>>;
  
  // Update user preferences
  updateUserPreferences(userId: string, preferences: Partial<UserPreferences>): Promise<ServiceResponse<User>>;
  
  // Change password
  changePassword(userId: string, passwordData: PasswordChangeData): Promise<ServiceResponse<void>>;
  
  // Delete user account
  deleteUserAccount(userId: string, confirmation: AccountDeletionData): Promise<ServiceResponse<void>>;
  
  // Get user booking history
  getUserBookingHistory(userId: string, filters?: BookingHistoryFilters): Promise<ServiceResponse<Booking[]>>;
}

interface PasswordChangeData {
  currentPassword: string;
  newPassword: string;
}

interface AccountDeletionData {
  password: string;
  confirmation: string;
  reason?: string;
}

interface BookingHistoryFilters {
  dateRange?: {
    start: Date;
    end: Date;
  };
  status?: BookingStatus[];
  sortBy?: 'date' | 'status' | 'amount';
  sortOrder?: 'asc' | 'desc';
  limit?: number;
  offset?: number;
}
```

## Amenity Service

### Interface Definition
```typescript
interface AmenityService {
  // Get all amenities
  getAllAmenities(): Promise<ServiceResponse<Amenity[]>>;
  
  // Get amenities by category
  getAmenitiesByCategory(category: AmenityCategory): Promise<ServiceResponse<Amenity[]>>;
  
  // Get amenity details
  getAmenityDetails(amenityId: string): Promise<ServiceResponse<Amenity>>;
  
  // Check amenity availability
  checkAmenityAvailability(
    amenityId: string, 
    dateTime: Date, 
    partySize?: number
  ): Promise<ServiceResponse<AmenityAvailabilityResult>>;
}

interface AmenityAvailabilityResult {
  amenityId: string;
  isAvailable: boolean;
  nextAvailableTime?: Date;
  waitingTime?: number; // minutes
  requiresReservation: boolean;
  capacityRemaining?: number;
}
```

## Property Service

### Interface Definition
```typescript
interface PropertyService {
  // Get property information
  getPropertyInfo(): Promise<ServiceResponse<Property>>;
  
  // Get nearby attractions
  getNearbyAttractions(radius?: number): Promise<ServiceResponse<Landmark[]>>;
  
  // Get weather information
  getWeatherInfo(dateRange?: DateRange): Promise<ServiceResponse<WeatherForecast>>;
  
  // Get cultural information
  getCulturalInfo(): Promise<ServiceResponse<CulturalInfo>>;
}

interface WeatherForecast {
  currentWeather: WeatherCondition;
  forecast: WeatherCondition[];
  averageTemperatures: {
    high: number;
    low: number;
  };
  bestVisitRecommendations: string[];
}

interface WeatherCondition {
  date: Date;
  temperature: {
    high: number;
    low: number;
  };
  humidity: number;
  precipitation: number;
  windSpeed: number;
  conditions: string;
  uvIndex: number;
}
```

## Common Service Types

### Service Response Wrapper
```typescript
interface ServiceResponse<T> {
  success: boolean;
  data?: T;
  error?: ServiceError;
  metadata?: ResponseMetadata;
}

interface ServiceError {
  code: string;
  message: string;
  details?: Record<string, any>;
  timestamp: Date;
  requestId: string;
}

interface ResponseMetadata {
  requestId: string;
  timestamp: Date;
  processingTime: number;
  pagination?: PaginationInfo;
}

interface PaginationInfo {
  page: number;
  limit: number;
  total: number;
  hasNext: boolean;
  hasPrevious: boolean;
}
```

### Error Handling Patterns
```typescript
// Standard error codes for consistent handling
enum ServiceErrorCode {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  NOT_FOUND = 'NOT_FOUND',
  UNAUTHORIZED = 'UNAUTHORIZED',
  FORBIDDEN = 'FORBIDDEN',
  CONFLICT = 'CONFLICT',
  RATE_LIMITED = 'RATE_LIMITED',
  SERVICE_UNAVAILABLE = 'SERVICE_UNAVAILABLE',
  INTERNAL_ERROR = 'INTERNAL_ERROR'
}

// Helper function for consistent error handling
function createServiceError(
  code: ServiceErrorCode,
  message: string,
  details?: Record<string, any>
): ServiceError {
  return {
    code,
    message,
    details,
    timestamp: new Date(),
    requestId: generateRequestId()
  };
}
```

## Mock Data Population

Each service includes comprehensive mock data:

### Room Service Mock Data
- 20+ diverse room types representing luxury Oahu hotel
- Realistic Hawaiian room names and descriptions
- Varied pricing reflecting seasonal demand
- Ocean view, garden view, and mountain view options

### Package Service Mock Data
- Romance packages: couples massage, sunset dinner, private beach
- Family packages: kids club, snorkeling, cultural tours
- Wellness packages: spa treatments, yoga, meditation
- Adventure packages: hiking, surfing, helicopter tours

### Amenity Service Mock Data
- Recreation: multiple pools, fitness center, tennis court
- Dining: restaurants, bars, room service, luau
- Business: meeting rooms, business center, conference facilities
- Wellness: spa, salon, wellness center

### User/Booking Mock Data
- Sample user accounts with varied preferences
- Historical bookings showing different scenarios
- Realistic booking patterns and seasonal variations

## Testing Strategy

Each service contract includes:
- Unit tests for all service methods
- Integration tests for service interactions
- Mock data validation tests
- Error handling verification
- Performance benchmarking for realistic response times

Contract tests ensure that future database implementations maintain compatibility with the defined interfaces.

**Status**: Phase 1 Service Contracts Complete - Ready for Quickstart Guide