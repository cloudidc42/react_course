# Part 29: React Hook Form - การจัดการ Form ขั้นสูง

**Step 771-810** | ระดับ: สูง | เวลาเรียน: 4-5 ชั่วโมง

---

## สารบัญ (Table of Contents)

1. [React Hook Form vs Controlled Forms](#react-hook-form-vs-controlled-forms)
2. [useForm, register, handleSubmit, watch](#useform-register-handlesubmit-watch)
3. [Validation พื้นฐาน](#validation-พื้นฐาน)
4. [Zod Schema Validation](#zod-schema-validation)
5. [Field Arrays](#field-arrays)
6. [Custom Input Components](#custom-input-components)
7. [File Upload](#file-upload)
8. [Multi-step Form](#multi-step-form)
9. [ตัวอย่าง Registration Form ครบ](#ตัวอย่าง-registration-form-ครบ)
10. [Quiz](#quiz)

---

## Step 771-773: React Hook Form vs Controlled Forms

### ปัญหาของ Controlled Forms

```jsx
// ❌ Controlled Form - re-render ทุก keystroke
function ContactForm() {
  const [name, setName] = useState('')
  const [email, setEmail] = useState('')
  const [phone, setPhone] = useState('')
  const [message, setMessage] = useState('')
  const [errors, setErrors] = useState({})

  // ต้อง validate เอง
  const validate = () => {
    const newErrors = {}
    if (!name) newErrors.name = 'กรุณากรอกชื่อ'
    if (!email) newErrors.email = 'กรุณากรอกอีเมล'
    // ...
    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  return (
    <form>
      <input value={name} onChange={e => setName(e.target.value)} />
      {errors.name && <span>{errors.name}</span>}
      {/* ทุก field ต้องการ state + error + onChange */}
    </form>
  )
}
```

### React Hook Form - เบากว่ามาก

```
ข้อดี React Hook Form:
✅ ไม่ re-render ขณะพิมพ์ (uncontrolled by default)
✅ Validation ง่ายกว่า
✅ Performance ดีกว่ามาก
✅ TypeScript support ดี
✅ Integration กับ UI libraries ง่าย
✅ Bundle size เล็ก (~13KB)
```

---

## Step 774-778: useForm, register, handleSubmit, watch

### การติดตั้ง

```bash
npm install react-hook-form
```

### API หลัก

```jsx
import { useForm } from 'react-hook-form'

function SimpleForm() {
  const {
    register,         // เชื่อม input กับ form
    handleSubmit,     // wrap submit handler
    watch,            // ดูค่า field แบบ realtime
    reset,            // reset form
    setValue,         // set ค่า programmatically
    getValues,        // อ่านค่าทั้งหมด
    formState: {
      errors,         // validation errors
      isSubmitting,   // true ขณะ submit
      isDirty,        // true ถ้า user แก้ form
      isValid,        // true ถ้า validation ผ่านทั้งหมด
      touchedFields,  // fields ที่ user เคย touch
    },
  } = useForm({
    defaultValues: {
      name: '',
      email: '',
      message: '',
    },
    mode: 'onChange',  // validate เมื่อ: 'onChange' | 'onBlur' | 'onSubmit'
  })

  const onSubmit = async (data) => {
    console.log(data)  // { name: '...', email: '...', message: '...' }
    await submitAPI(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          {...register('name', {
            required: 'กรุณากรอกชื่อ',
            minLength: { value: 2, message: 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร' },
          })}
          placeholder="ชื่อ"
        />
        {errors.name && <span style={{ color: 'red' }}>{errors.name.message}</span>}
      </div>

      <div>
        <input
          {...register('email', {
            required: 'กรุณากรอกอีเมล',
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: 'อีเมลไม่ถูกต้อง',
            },
          })}
          placeholder="อีเมล"
        />
        {errors.email && <span style={{ color: 'red' }}>{errors.email.message}</span>}
      </div>

      <div>
        <textarea
          {...register('message', {
            required: 'กรุณากรอกข้อความ',
            minLength: { value: 10, message: 'ข้อความต้องมีอย่างน้อย 10 ตัวอักษร' },
            maxLength: { value: 500, message: 'ข้อความต้องไม่เกิน 500 ตัวอักษร' },
          })}
          placeholder="ข้อความ"
        />
        {errors.message && <span style={{ color: 'red' }}>{errors.message.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'กำลังส่ง...' : 'ส่งข้อความ'}
      </button>
    </form>
  )
}
```

### watch - ดูค่า realtime

```jsx
function PriceCalculator() {
  const { register, watch } = useForm()
  
  const quantity = watch('quantity', 0)
  const price = watch('price', 0)
  const total = quantity * price

  return (
    <form>
      <input {...register('quantity')} type="number" placeholder="จำนวน" />
      <input {...register('price')} type="number" placeholder="ราคา" />
      <p>รวม: ฿{total}</p>
    </form>
  )
}
```

---

## Step 779-784: Zod Schema Validation

### การติดตั้ง

```bash
npm install zod @hookform/resolvers
```

### สร้าง Zod Schema

```jsx
import { z } from 'zod'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'

// กำหนด schema
const registrationSchema = z.object({
  firstName: z.string()
    .min(2, 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร')
    .max(50, 'ชื่อต้องไม่เกิน 50 ตัวอักษร'),
  
  lastName: z.string()
    .min(2, 'นามสกุลต้องมีอย่างน้อย 2 ตัวอักษร'),
  
  email: z.string()
    .email('อีเมลไม่ถูกต้อง'),
  
  phone: z.string()
    .regex(/^0[0-9]{9}$/, 'เบอร์โทรต้องเป็นตัวเลข 10 หลักขึ้นต้นด้วย 0'),
  
  password: z.string()
    .min(8, 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
    .regex(/[A-Z]/, 'ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว')
    .regex(/[0-9]/, 'ต้องมีตัวเลขอย่างน้อย 1 ตัว'),
  
  confirmPassword: z.string(),
  
  age: z.number()
    .min(18, 'ต้องมีอายุอย่างน้อย 18 ปี')
    .max(100, 'อายุไม่ถูกต้อง'),
  
  role: z.enum(['user', 'admin', 'editor']),
  
  termsAccepted: z.boolean()
    .refine(val => val === true, 'กรุณายอมรับเงื่อนไขการใช้บริการ'),
}).refine(data => data.password === data.confirmPassword, {
  message: 'รหัสผ่านไม่ตรงกัน',
  path: ['confirmPassword'],
})

type RegistrationFormData = z.infer<typeof registrationSchema>

function RegistrationForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
    resolver: zodResolver(registrationSchema),
    defaultValues: {
      role: 'user',
      termsAccepted: false,
    },
  })

  const onSubmit = async (data) => {
    await registerUser(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('firstName')} placeholder="ชื่อ" />
      {errors.firstName && <p>{errors.firstName.message}</p>}
      
      {/* ... other fields */}
      
      <button type="submit" disabled={isSubmitting}>สมัครสมาชิก</button>
    </form>
  )
}
```

---

## Step 785-789: Field Arrays

```jsx
import { useFieldArray, useForm } from 'react-hook-form'

function OrderForm() {
  const { register, control, handleSubmit, watch } = useForm({
    defaultValues: {
      customerName: '',
      items: [{ productId: '', quantity: 1, note: '' }],
    },
  })

  const { fields, append, remove, move, insert } = useFieldArray({
    control,
    name: 'items',
  })

  const onSubmit = (data) => console.log(data)

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('customerName')} placeholder="ชื่อลูกค้า" />

      <h3>รายการสินค้า</h3>

      {fields.map((field, index) => (
        <div key={field.id} style={{ display: 'flex', gap: '0.5rem', marginBottom: '0.5rem' }}>
          <input
            {...register(`items.${index}.productId`)}
            placeholder="รหัสสินค้า"
          />
          <input
            {...register(`items.${index}.quantity`, { valueAsNumber: true })}
            type="number"
            min={1}
            placeholder="จำนวน"
          />
          <input
            {...register(`items.${index}.note`)}
            placeholder="หมายเหตุ"
          />
          <button
            type="button"
            onClick={() => remove(index)}
            disabled={fields.length === 1}
          >
            ลบ
          </button>
        </div>
      ))}

      <button
        type="button"
        onClick={() => append({ productId: '', quantity: 1, note: '' })}
      >
        + เพิ่มรายการ
      </button>

      <button type="submit">สั่งซื้อ</button>
    </form>
  )
}
```

---

## Step 790-793: Custom Input Components

### Controller API

```jsx
import { Controller, useForm } from 'react-hook-form'
import Select from 'react-select'
import DatePicker from 'react-datepicker'

function AdvancedForm() {
  const { control, handleSubmit } = useForm()

  return (
    <form onSubmit={handleSubmit(console.log)}>
      {/* react-select */}
      <Controller
        name="category"
        control={control}
        rules={{ required: 'กรุณาเลือกหมวดหมู่' }}
        render={({ field, fieldState }) => (
          <div>
            <Select
              {...field}
              options={[
                { value: 'electronics', label: 'อิเล็กทรอนิกส์' },
                { value: 'clothing', label: 'เสื้อผ้า' },
              ]}
              placeholder="เลือกหมวดหมู่"
            />
            {fieldState.error && <p>{fieldState.error.message}</p>}
          </div>
        )}
      />

      {/* Date Picker */}
      <Controller
        name="birthDate"
        control={control}
        rules={{ required: 'กรุณาเลือกวันเกิด' }}
        render={({ field }) => (
          <DatePicker
            selected={field.value}
            onChange={field.onChange}
            dateFormat="dd/MM/yyyy"
            placeholderText="วันเกิด"
          />
        )}
      />

      <button type="submit">บันทึก</button>
    </form>
  )
}
```

### Custom Reusable Input

```jsx
// components/FormInput.jsx
import { useFormContext } from 'react-hook-form'

function FormInput({ name, label, type = 'text', placeholder, rules, ...props }) {
  const {
    register,
    formState: { errors },
  } = useFormContext()

  const error = errors[name]

  return (
    <div style={{ marginBottom: '1rem' }}>
      {label && (
        <label htmlFor={name} style={{ display: 'block', marginBottom: '0.25rem', fontWeight: '500' }}>
          {label}
          {rules?.required && <span style={{ color: 'red' }}> *</span>}
        </label>
      )}
      <input
        id={name}
        type={type}
        placeholder={placeholder}
        style={{
          width: '100%',
          padding: '0.5rem',
          border: `1px solid ${error ? '#e53e3e' : '#e2e8f0'}`,
          borderRadius: '4px',
          outline: 'none',
        }}
        {...register(name, rules)}
        {...props}
      />
      {error && (
        <p style={{ color: '#e53e3e', fontSize: '0.875rem', marginTop: '0.25rem' }}>
          {error.message}
        </p>
      )}
    </div>
  )
}

// ใช้กับ FormProvider
import { FormProvider, useForm } from 'react-hook-form'

function MyForm() {
  const methods = useForm()

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(console.log)}>
        <FormInput name="email" label="อีเมล" type="email" rules={{ required: 'กรุณากรอกอีเมล' }} />
        <FormInput name="password" label="รหัสผ่าน" type="password" rules={{ required: 'กรุณากรอกรหัสผ่าน' }} />
        <button type="submit">บันทึก</button>
      </form>
    </FormProvider>
  )
}
```

---

## Step 794-797: File Upload

```jsx
import { useForm } from 'react-hook-form'

function ProfileForm() {
  const [preview, setPreview] = useState(null)
  const { register, handleSubmit, formState: { errors } } = useForm()

  const handleFileChange = (e) => {
    const file = e.target.files[0]
    if (file) {
      const reader = new FileReader()
      reader.onload = () => setPreview(reader.result)
      reader.readAsDataURL(file)
    }
  }

  const onSubmit = async (data) => {
    const formData = new FormData()
    formData.append('name', data.name)
    formData.append('avatar', data.avatar[0])  // FileList[0]

    await fetch('/api/profile', {
      method: 'POST',
      body: formData,  // ไม่ต้อง Content-Type header เมื่อใช้ FormData
    })
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} placeholder="ชื่อ" />

      <div>
        {preview && (
          <img src={preview} alt="preview" style={{ width: 100, height: 100, objectFit: 'cover', borderRadius: '50%' }} />
        )}
        <input
          type="file"
          accept="image/*"
          {...register('avatar', {
            required: 'กรุณาเลือกรูปภาพ',
            validate: {
              size: (files) => files[0]?.size < 5 * 1024 * 1024 || 'ไฟล์ต้องไม่เกิน 5MB',
              type: (files) => ['image/jpeg', 'image/png', 'image/webp'].includes(files[0]?.type) || 'รองรับเฉพาะ JPG, PNG, WebP',
            },
            onChange: handleFileChange,
          })}
        />
        {errors.avatar && <p style={{ color: 'red' }}>{errors.avatar.message}</p>}
      </div>

      <button type="submit">บันทึก</button>
    </form>
  )
}
```

---

## Step 798-803: Multi-step Form

```jsx
import { useForm, FormProvider } from 'react-hook-form'
import { useState } from 'react'
import { z } from 'zod'
import { zodResolver } from '@hookform/resolvers/zod'

// Schemas สำหรับแต่ละ step
const step1Schema = z.object({
  firstName: z.string().min(2),
  lastName: z.string().min(2),
  email: z.string().email(),
})

const step2Schema = z.object({
  phone: z.string().regex(/^0[0-9]{9}$/),
  address: z.string().min(10),
  city: z.string().min(2),
})

const step3Schema = z.object({
  username: z.string().min(3).max(20),
  password: z.string().min(8),
  confirmPassword: z.string(),
}).refine(d => d.password === d.confirmPassword, {
  message: 'รหัสผ่านไม่ตรงกัน',
  path: ['confirmPassword'],
})

const STEPS = [
  { title: 'ข้อมูลส่วนตัว', schema: step1Schema },
  { title: 'ที่อยู่', schema: step2Schema },
  { title: 'ข้อมูลบัญชี', schema: step3Schema },
]

function MultiStepRegistration() {
  const [currentStep, setCurrentStep] = useState(0)
  const [allData, setAllData] = useState({})
  const [isComplete, setIsComplete] = useState(false)

  const methods = useForm({
    resolver: zodResolver(STEPS[currentStep].schema),
    defaultValues: allData,
  })

  const goNext = methods.handleSubmit((data) => {
    setAllData(prev => ({ ...prev, ...data }))

    if (currentStep < STEPS.length - 1) {
      setCurrentStep(prev => prev + 1)
      methods.reset({ ...allData, ...data })
    } else {
      // Submit ทั้งหมด
      handleFinalSubmit({ ...allData, ...data })
    }
  })

  const goBack = () => {
    setCurrentStep(prev => prev - 1)
    methods.reset(allData)
  }

  const handleFinalSubmit = async (data) => {
    await registerUser(data)
    setIsComplete(true)
  }

  if (isComplete) {
    return <div>สมัครสมาชิกสำเร็จ! 🎉</div>
  }

  return (
    <div style={{ maxWidth: '500px', margin: '0 auto', padding: '2rem' }}>
      {/* Progress Bar */}
      <div style={{ marginBottom: '2rem' }}>
        <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '0.5rem' }}>
          {STEPS.map((step, i) => (
            <span key={i} style={{
              color: i <= currentStep ? '#007bff' : '#a0aec0',
              fontWeight: i === currentStep ? 'bold' : 'normal',
            }}>
              {i + 1}. {step.title}
            </span>
          ))}
        </div>
        <div style={{ height: '4px', backgroundColor: '#e2e8f0', borderRadius: '2px' }}>
          <div style={{
            height: '100%',
            width: `${((currentStep + 1) / STEPS.length) * 100}%`,
            backgroundColor: '#007bff',
            borderRadius: '2px',
            transition: 'width 0.3s ease',
          }} />
        </div>
      </div>

      <h2>{STEPS[currentStep].title}</h2>

      <FormProvider {...methods}>
        <form onSubmit={goNext}>
          {currentStep === 0 && <Step1Fields />}
          {currentStep === 1 && <Step2Fields />}
          {currentStep === 2 && <Step3Fields />}

          <div style={{ display: 'flex', justifyContent: 'space-between', marginTop: '1.5rem' }}>
            <button
              type="button"
              onClick={goBack}
              disabled={currentStep === 0}
              style={{ opacity: currentStep === 0 ? 0.5 : 1 }}
            >
              ย้อนกลับ
            </button>
            <button type="submit" style={{ backgroundColor: '#007bff', color: 'white', padding: '0.5rem 1.5rem', border: 'none', borderRadius: '4px' }}>
              {currentStep === STEPS.length - 1 ? 'สมัครสมาชิก' : 'ถัดไป'}
            </button>
          </div>
        </form>
      </FormProvider>
    </div>
  )
}

function Step1Fields() {
  const { register, formState: { errors } } = useFormContext()
  return (
    <>
      <div>
        <input {...register('firstName')} placeholder="ชื่อ" />
        {errors.firstName && <p style={{ color: 'red' }}>{errors.firstName.message}</p>}
      </div>
      <div>
        <input {...register('lastName')} placeholder="นามสกุล" />
        {errors.lastName && <p style={{ color: 'red' }}>{errors.lastName.message}</p>}
      </div>
      <div>
        <input {...register('email')} placeholder="อีเมล" type="email" />
        {errors.email && <p style={{ color: 'red' }}>{errors.email.message}</p>}
      </div>
    </>
  )
}
```

---

## Step 804-810: ตัวอย่าง Registration Form ครบสมบูรณ์

```jsx
// forms/RegistrationForm.jsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { useState } from 'react'
import { useNavigate } from 'react-router-dom'

const schema = z.object({
  firstName: z.string().min(2, 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร'),
  lastName: z.string().min(2, 'นามสกุลต้องมีอย่างน้อย 2 ตัวอักษร'),
  email: z.string().email('อีเมลไม่ถูกต้อง'),
  phone: z.string().regex(/^0\d{9}$/, 'เบอร์โทรต้อง 10 หลัก'),
  password: z.string()
    .min(8, 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร')
    .regex(/[A-Z]/, 'ต้องมีตัวพิมพ์ใหญ่')
    .regex(/\d/, 'ต้องมีตัวเลข'),
  confirmPassword: z.string(),
  gender: z.enum(['male', 'female', 'other']),
  birthDate: z.string().refine(val => {
    const age = new Date().getFullYear() - new Date(val).getFullYear()
    return age >= 18
  }, 'ต้องมีอายุอย่างน้อย 18 ปี'),
  terms: z.boolean().refine(v => v, 'กรุณายอมรับเงื่อนไข'),
  newsletter: z.boolean().optional(),
}).refine(d => d.password === d.confirmPassword, {
  message: 'รหัสผ่านไม่ตรงกัน',
  path: ['confirmPassword'],
})

function RegistrationForm() {
  const [showPassword, setShowPassword] = useState(false)
  const navigate = useNavigate()

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isSubmitSuccessful },
    setError,
    reset,
  } = useForm({
    resolver: zodResolver(schema),
  })

  const onSubmit = async (data) => {
    try {
      await authService.register(data)
      navigate('/login', { state: { message: 'สมัครสมาชิกสำเร็จ กรุณาเข้าสู่ระบบ' } })
    } catch (error) {
      if (error.response?.status === 422) {
        const serverErrors = error.response.data.errors
        Object.entries(serverErrors).forEach(([field, message]) => {
          setError(field, { message: Array.isArray(message) ? message[0] : message })
        })
      } else {
        setError('root', { message: 'เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง' })
      }
    }
  }

  const inputStyle = (hasError) => ({
    width: '100%',
    padding: '0.75rem',
    border: `1px solid ${hasError ? '#e53e3e' : '#e2e8f0'}`,
    borderRadius: '8px',
    fontSize: '1rem',
    outline: 'none',
    boxSizing: 'border-box',
  })

  return (
    <div style={{ maxWidth: '500px', margin: '2rem auto', padding: '2rem', boxShadow: '0 4px 6px rgba(0,0,0,0.1)', borderRadius: '12px' }}>
      <h2 style={{ textAlign: 'center', marginBottom: '2rem' }}>สมัครสมาชิก</h2>

      {errors.root && (
        <div style={{ backgroundColor: '#fff5f5', border: '1px solid #fc8181', padding: '1rem', borderRadius: '8px', marginBottom: '1rem' }}>
          {errors.root.message}
        </div>
      )}

      <form onSubmit={handleSubmit(onSubmit)}>
        <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '1rem', marginBottom: '1rem' }}>
          <div>
            <label>ชื่อ *</label>
            <input {...register('firstName')} style={inputStyle(errors.firstName)} placeholder="ชื่อ" />
            {errors.firstName && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.firstName.message}</p>}
          </div>
          <div>
            <label>นามสกุล *</label>
            <input {...register('lastName')} style={inputStyle(errors.lastName)} placeholder="นามสกุล" />
            {errors.lastName && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.lastName.message}</p>}
          </div>
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label>อีเมล *</label>
          <input {...register('email')} type="email" style={inputStyle(errors.email)} placeholder="example@email.com" />
          {errors.email && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.email.message}</p>}
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label>เบอร์โทรศัพท์ *</label>
          <input {...register('phone')} type="tel" style={inputStyle(errors.phone)} placeholder="0812345678" />
          {errors.phone && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.phone.message}</p>}
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label>รหัสผ่าน *</label>
          <div style={{ position: 'relative' }}>
            <input
              {...register('password')}
              type={showPassword ? 'text' : 'password'}
              style={inputStyle(errors.password)}
              placeholder="อย่างน้อย 8 ตัวอักษร"
            />
            <button
              type="button"
              onClick={() => setShowPassword(!showPassword)}
              style={{ position: 'absolute', right: '0.75rem', top: '50%', transform: 'translateY(-50%)', background: 'none', border: 'none', cursor: 'pointer' }}
            >
              {showPassword ? '🙈' : '👁'}
            </button>
          </div>
          {errors.password && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.password.message}</p>}
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label>ยืนยันรหัสผ่าน *</label>
          <input {...register('confirmPassword')} type="password" style={inputStyle(errors.confirmPassword)} />
          {errors.confirmPassword && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.confirmPassword.message}</p>}
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label>เพศ *</label>
          <select {...register('gender')} style={inputStyle(errors.gender)}>
            <option value="">เลือกเพศ</option>
            <option value="male">ชาย</option>
            <option value="female">หญิง</option>
            <option value="other">อื่นๆ</option>
          </select>
          {errors.gender && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.gender.message}</p>}
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label>วันเกิด *</label>
          <input {...register('birthDate')} type="date" style={inputStyle(errors.birthDate)} />
          {errors.birthDate && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.birthDate.message}</p>}
        </div>

        <div style={{ marginBottom: '1rem' }}>
          <label style={{ display: 'flex', alignItems: 'center', gap: '0.5rem' }}>
            <input type="checkbox" {...register('terms')} />
            ฉันยอมรับ <a href="/terms" target="_blank">เงื่อนไขการใช้บริการ</a> *
          </label>
          {errors.terms && <p style={{ color: '#e53e3e', fontSize: '0.8rem' }}>{errors.terms.message}</p>}
        </div>

        <div style={{ marginBottom: '1.5rem' }}>
          <label style={{ display: 'flex', alignItems: 'center', gap: '0.5rem' }}>
            <input type="checkbox" {...register('newsletter')} />
            รับข่าวสารและโปรโมชั่น
          </label>
        </div>

        <button
          type="submit"
          disabled={isSubmitting}
          style={{
            width: '100%',
            padding: '0.75rem',
            backgroundColor: '#007bff',
            color: 'white',
            border: 'none',
            borderRadius: '8px',
            fontSize: '1rem',
            cursor: isSubmitting ? 'not-allowed' : 'pointer',
            opacity: isSubmitting ? 0.7 : 1,
          }}
        >
          {isSubmitting ? 'กำลังสมัครสมาชิก...' : 'สมัครสมาชิก'}
        </button>
      </form>
    </div>
  )
}

export default RegistrationForm
```

---

## Quiz - Part 29

**ข้อ 1**: ข้อดีหลักของ React Hook Form เทียบกับ Controlled Forms?
- a) รองรับ TypeScript มากกว่า
- b) Performance ดีกว่า ไม่ re-render ทุก keystroke
- c) มี built-in UI components
- d) API ง่ายกว่า

**ข้อ 2**: `useFieldArray` ใช้ทำอะไร?
- a) Validate array fields
- b) จัดการ dynamic list ของ form fields
- c) Upload ไฟล์หลายๆ อัน
- d) Reset form

**ข้อ 3**: Zod `refine` method ใช้ทำอะไร?
- a) Transform ข้อมูล
- b) เพิ่ม custom validation logic
- c) กำหนด default values
- d) Format output

**ข้อ 4**: `Controller` ใน React Hook Form ใช้เมื่อใด?
- a) เสมอ
- b) กับ custom/third-party components ที่ไม่รองรับ register
- c) สำหรับ textarea เท่านั้น
- d) สำหรับ file upload เท่านั้น

**คำตอบ**: 1-b, 2-b, 3-b, 4-b

---

## สรุป Part 29

ใน Part นี้คุณได้เรียนรู้:
- ✅ React Hook Form vs Controlled Forms
- ✅ useForm, register, handleSubmit, watch
- ✅ Validation (built-in และ Zod schema)
- ✅ Field Arrays (dynamic forms)
- ✅ Custom Input Components กับ Controller
- ✅ File Upload
- ✅ Multi-step Form
- ✅ Registration Form ครบสมบูรณ์

---

## Part ถัดไป

➡️ **[Part 30: TypeScript + React](./part-30-typescript-with-react.md)**
- TypeScript basics สำหรับ React
- Component Props Types
- Generic Components
- Utility Types
